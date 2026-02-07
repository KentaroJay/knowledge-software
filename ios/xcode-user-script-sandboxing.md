# Xcode User Script Sandboxing and React Native Builds

Understanding `ENABLE_USER_SCRIPT_SANDBOXING`, why it breaks React Native iOS builds, and how to fix it.

## 📦 Quick Summary

| Aspect | Details |
|--------|---------|
| **Setting** | `ENABLE_USER_SCRIPT_SANDBOXING` in Xcode build settings |
| **Introduced** | Xcode 14.3 (default `YES` for new projects in Xcode 15+) |
| **Impact** | Breaks React Native builds by blocking shell script access to `node_modules` |
| **Fix** | Set to `NO` in project-level build configurations |
| **Location** | `ios/<AppName>.xcodeproj/project.pbxproj` |
| **Affects** | Build phase scripts: JS bundling, CocoaPods phases, codegen |

---

## 🔍 Context

When you run `npm run ios` (or `npx react-native run-ios`), the React Native CLI triggers an Xcode build. Part of that build executes several shell scripts in **build phases** — scripts that bundle your JavaScript, copy assets, check pod manifests, and more.

Xcode 14.3 introduced a security feature called **User Script Sandboxing** that, when enabled, restricts these build phase scripts to a narrow sandbox. They can no longer freely read from or write to the filesystem. For typical iOS-only projects this is fine, but React Native projects rely heavily on build scripts that reach outside the project directory into `node_modules`, run Node.js, and execute tools like Metro and Hermes.

When `ENABLE_USER_SCRIPT_SANDBOXING` is `YES`, the build fails because these scripts are denied file access.

---

## 🧩 What Build Phase Scripts Does React Native Use?

A React Native iOS project has several shell script build phases. You can see these in the `project.pbxproj` file under `PBXShellScriptBuildPhase`:

### 1. Bundle React Native code and images

This is the critical one. It runs `react-native-xcode.sh`, which:
- Sources environment variables (Node path, Hermes settings)
- Invokes Node.js to run the Metro bundler
- Produces `main.jsbundle` for release builds
- Copies static assets (images, fonts) into the app bundle

```bash
# Simplified version of what this phase does
set -e
WITH_ENVIRONMENT="$REACT_NATIVE_PATH/scripts/xcode/with-environment.sh"
REACT_NATIVE_XCODE="$REACT_NATIVE_PATH/scripts/react-native-xcode.sh"
/bin/sh -c "\"$WITH_ENVIRONMENT\" \"$REACT_NATIVE_XCODE\""
```

This script needs to:
- Read `$REACT_NATIVE_PATH` which points to `node_modules/react-native`
- Execute Node.js from wherever it's installed (e.g., `/Users/<name>/.nvm/versions/node/...`)
- Read your project's `index.js` entry point
- Write the bundle output into the build products directory

All of this fails under sandboxing.

### 2. [CP] Check Pods Manifest.lock

A CocoaPods script that compares `Podfile.lock` with the Pods manifest to ensure pods are in sync:

```bash
diff "${PODS_PODFILE_DIR_PATH}/Podfile.lock" "${PODS_ROOT}/Manifest.lock" > /dev/null
```

### 3. [CP] Embed Pods Frameworks

Copies dynamic frameworks from pods into the app bundle.

### 4. [CP] Copy Pods Resources

Copies pod resources (images, xibs, storyboards) into the app bundle.

All of these scripts need filesystem access outside the sandbox boundary.

---

## 💥 What the Failure Looks Like

When `ENABLE_USER_SCRIPT_SANDBOXING = YES`, you'll see errors like:

```
Sandbox: bash(<PID>) deny(1) file-read-data /Users/<name>/Projects/MyApp/node_modules/react-native/scripts/xcode/with-environment.sh

PhaseScriptExecution [CP]\ Check\ Pods\ Manifest.lock
    Sandbox: bash(<PID>) deny(1) file-read-data .../Pods/Manifest.lock

Script phase "Bundle React Native code and images" failed with a nonzero exit code
```

The key indicator is `deny(1) file-read-data` — Xcode's sandbox is blocking the script from reading files it needs.

---

## 🔧 The Fix

### Where the setting lives

The setting is in `ios/<AppName>.xcodeproj/project.pbxproj`, inside the **project-level** `XCBuildConfiguration` sections (not the target-level ones).

A typical `project.pbxproj` has four `XCBuildConfiguration` blocks:

```
XCBuildConfiguration sections:
├── Target-level Debug    (inherits from project)
├── Target-level Release  (inherits from project)
├── Project-level Debug   ← ENABLE_USER_SCRIPT_SANDBOXING lives here
└── Project-level Release ← and here
```

### How to identify project-level vs target-level configs

The project-level configs are referenced by the `PBXProject` section's `buildConfigurationList`. The target-level configs are referenced by `PBXNativeTarget`. In practice, look for the configuration blocks that contain settings like `CLANG_CXX_LANGUAGE_STANDARD`, `GCC_C_LANGUAGE_STANDARD`, `SDKROOT` — these are project-level. The target ones contain `INFOPLIST_FILE`, `PRODUCT_BUNDLE_IDENTIFIER`, `PRODUCT_NAME`.

### The change

Find the project-level Debug and Release configurations and change:

```diff
- ENABLE_USER_SCRIPT_SANDBOXING = YES;
+ ENABLE_USER_SCRIPT_SANDBOXING = NO;
```

This needs to be done in **both** the Debug and Release project-level configurations.

### Alternative: Fix via Xcode UI

1. Open `<AppName>.xcworkspace` in Xcode
2. Click on the **project** (top-level, blue icon) in the navigator — not the target
3. Select **Build Settings** tab
4. Search for "User Script Sandboxing"
5. Set it to **No** for both Debug and Release

---

## 🏗️ Understanding the project.pbxproj Structure

The `project.pbxproj` file is the heart of an Xcode project. Knowing how to navigate it helps when debugging build issues:

```
project.pbxproj
├── PBXBuildFile section       — Files included in build
├── PBXFileReference section   — All files known to the project
├── PBXFrameworksBuildPhase    — Linked frameworks
├── PBXGroup section           — File tree structure (what you see in Xcode)
├── PBXNativeTarget section    — Build targets (your app)
├── PBXProject section         — Project-wide settings
├── PBXResourcesBuildPhase     — Resources copied into the bundle
├── PBXShellScriptBuildPhase   — Build phase shell scripts ← where RN scripts live
├── PBXSourcesBuildPhase       — Source files to compile
├── XCBuildConfiguration       — Debug/Release settings ← where the fix goes
└── XCConfigurationList        — Groups configurations together
```

The `XCBuildConfiguration` entries come in pairs (Debug + Release) and there are two pairs: one for the project and one for each target. The project-level pair sets defaults that targets inherit (unless overridden).

---

## 🎯 Real-World Example: HabitTracker

When building the HabitTracker app (React Native 0.83.1), `npm run ios` failed. The `project.pbxproj` had this in both project-level configurations:

```
ENABLE_USER_SCRIPT_SANDBOXING = YES;
```

The fix was changing both occurrences to `NO`. No other changes were needed — the TypeScript compiled fine, the JS bundle built fine, the Podfile was correct. It was purely the Xcode sandboxing setting blocking the build scripts.

### Diagnosis process

1. **Run `npm run ios`** — got `spawnSync xcodebuild ENOENT` (no Xcode on the machine), but on a Mac with Xcode the error would be the sandbox `deny` messages
2. **Check TypeScript** — `npx tsc --noEmit` passed clean
3. **Check JS bundle** — `npx react-native bundle --platform ios ...` succeeded
4. **Check Podfile** — standard React Native configuration, nothing unusual
5. **Check AppDelegate.swift** — correct imports and setup for RN 0.83.1
6. **Check project.pbxproj** — found `ENABLE_USER_SCRIPT_SANDBOXING = YES` in project-level build configs

The fact that TypeScript and Metro bundling worked fine narrowed the problem to the native Xcode build pipeline, which led to examining the `project.pbxproj` settings.

---

## 🔄 Why Does This Happen?

### New project defaults

Starting with Xcode 15, new projects default to `ENABLE_USER_SCRIPT_SANDBOXING = YES`. If you create a React Native project using a tool or template that generates a fresh Xcode project, it may pick up this default.

### Xcode upgrades

Opening an older project in a newer version of Xcode can trigger an "upgrade" that flips this setting to `YES`. Xcode may suggest or automatically apply "recommended settings" that include enabling script sandboxing.

### React Native's mitigation

React Native's `react_native_post_install` hook in the Podfile does set `ENABLE_USER_SCRIPT_SANDBOXING = NO` for pod targets during `pod install`. However, this only affects pod targets — not the main project configuration. If the project-level setting is `YES`, the app target's build phases still run sandboxed.

---

## 🚨 Common Pitfalls

- **Don't confuse project-level and target-level configs.** The setting needs to be `NO` at the project level. Target-level configs inherit from the project, so if you only change it at the target level, other scripts (like CocoaPods phases) may still fail.

- **Watch for Xcode "recommended settings" prompts.** After fixing this, Xcode may pop up a dialog suggesting you re-enable sandboxing. Decline it, or you'll be back to square one.

- **This isn't a security risk for development.** User script sandboxing is a defense-in-depth measure for build scripts. Disabling it for a React Native project is standard practice and doesn't affect the runtime security of your app.

- **The setting may reappear after `pod install`.** Depending on your CocoaPods version and React Native version, running `pod install` may regenerate parts of the project file. Verify the setting after any pod operations.

- **Don't blindly set all settings to NO.** Only `ENABLE_USER_SCRIPT_SANDBOXING` needs to be `NO`. Other security-related settings (like `ENABLE_HARDENED_RUNTIME`) should be left at their defaults.

---

## 🎓 Key Takeaways

1. `ENABLE_USER_SCRIPT_SANDBOXING = YES` blocks React Native's build phase scripts from accessing `node_modules`, Node.js, and other filesystem paths they need
2. The fix is setting it to `NO` in the **project-level** Debug and Release configurations in `project.pbxproj`
3. This is a common issue when using Xcode 15+ with React Native, because new projects default to sandboxing enabled
4. Always check `project.pbxproj` when you get mysterious build failures that aren't related to your JavaScript or TypeScript code
5. The diagnostic approach is: verify TS compiles → verify JS bundles → check native config files

---

## 🔗 Related Topics

- See [npm vs CocoaPods](../react-native/npm-vs-cocoapods.md) for understanding the two-layer dependency system in React Native

---

## 📚 Further Reading

- [React Native Environment Setup](https://reactnative.dev/docs/set-up-your-environment)
- [Apple: Build settings reference — ENABLE_USER_SCRIPT_SANDBOXING](https://developer.apple.com/documentation/xcode/build-settings-reference)
- [React Native GitHub: Script sandboxing issues](https://github.com/facebook/react-native/issues)
- [CocoaPods: Xcode 15 compatibility](https://blog.cocoapods.org/)

---

*Created: February 2026*
*Project Context: HabitTracker app (React Native 0.83.1, Xcode build fix)*
