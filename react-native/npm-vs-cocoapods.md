# npm vs CocoaPods in React Native

Understanding the difference between JavaScript dependencies (npm) and native iOS dependencies (CocoaPods).

## 📦 Quick Summary

| Aspect | npm | CocoaPods |
|--------|-----|-----------|
| **Purpose** | JavaScript packages | Native iOS libraries |
| **Language** | JavaScript/TypeScript | Objective-C/Swift/C++ |
| **Location** | `node_modules/` | `ios/Pods/` |
| **Platform** | Cross-platform | iOS only |
| **Config File** | `package.json` | `Podfile` |
| **Lock File** | `package-lock.json` | `Podfile.lock` |
| **Command** | `npm install` | `pod install` |

---

## 📦 npm (Node Package Manager)

### What It Installs
- JavaScript/TypeScript packages
- React components
- Business logic libraries
- Cross-platform APIs

### Where It Installs
```
node_modules/
├── react/
├── react-native/
├── @react-navigation/
└── [other JS packages]
```

### Example Installation
```bash
npm install @react-navigation/native
npm install @react-native-async-storage/async-storage
npm install react-native-gesture-handler
```

### What You Get
- JavaScript code that runs in the React Native JavaScript engine
- React components written in TypeScript/JavaScript
- Business logic, state management, UI code
- Cross-platform APIs (work on iOS and Android)

---

## 🍎 CocoaPods (iOS Dependency Manager)

### What It Installs
- Native iOS libraries
- Objective-C/Swift implementations
- iOS SDK integrations
- Native UI components

### Where It Installs
```
ios/
├── Pods/
│   ├── React-Core/
│   ├── RNCAsyncStorage/
│   ├── RNGestureHandler/
│   └── [83+ other pods]
└── HabitTracker.xcworkspace  # Generated
```

### Example Installation
```bash
cd ios
pod install
cd ..
```

### What You Get
- Native iOS code that interfaces with iOS SDK
- Platform-specific implementations
- Access to iOS system features (camera, file system, etc.)
- Compiled native libraries

---

## 🔗 How They Work Together

React Native apps have **two layers** that communicate via a bridge:

```
┌─────────────────────────────────────┐
│   JavaScript Layer (npm)            │
│   ─────────────────────────         │
│   • Your React components           │
│   • React Navigation                │
│   • AsyncStorage JavaScript API     │
│   • Business logic                  │
│   • State management                │
└──────────────┬──────────────────────┘
               │
               │ React Native Bridge
               │ (Serialization/Deserialization)
               │
┌──────────────▼──────────────────────┐
│   Native Layer (CocoaPods)          │
│   ──────────────────────            │
│   • iOS implementation              │
│   • Access to iOS SDK               │
│   • Native UI components            │
│   • File system, camera, etc.       │
│   • UIKit, CoreData, etc.           │
└─────────────────────────────────────┘
```

---

## 🎯 Real-World Examples

### 1. AsyncStorage

**JavaScript Layer (npm):**
```javascript
// What you write in your code
import AsyncStorage from '@react-native-async-storage/async-storage';

// JavaScript API
await AsyncStorage.setItem('habits', JSON.stringify(habits));
const data = await AsyncStorage.getItem('habits');
```

**Native Layer (CocoaPods):**
- Pod: `RNCAsyncStorage`
- Native iOS code that actually writes to iOS file system
- Uses iOS `UserDefaults` or file storage
- Written in Objective-C

**Flow:**
1. You call `AsyncStorage.setItem()` in JavaScript
2. React Native Bridge serializes the data
3. Native iOS code receives the call
4. iOS writes to the file system
5. Result is sent back through the bridge
6. JavaScript Promise resolves

---

### 2. React Navigation

**JavaScript Layer (npm):**
```javascript
import {NavigationContainer} from '@react-navigation/native';
import {createBottomTabNavigator} from '@react-navigation/bottom-tabs';

// JavaScript navigation logic
const Tab = createBottomTabNavigator();

function App() {
  return (
    <NavigationContainer>
      <Tab.Navigator>
        <Tab.Screen name="Home" component={HomeScreen} />
      </Tab.Navigator>
    </NavigationContainer>
  );
}
```

**Native Layer (CocoaPods):**
- Pod: `react-native-screens`
- Pod: `RNGestureHandler`
- Native iOS screen optimizations
- Uses iOS `UIViewController` behind the scenes
- Native navigation animations

**Why Native Code Needed:**
- JavaScript can't create native iOS view controllers
- Native code provides smooth 60fps animations
- Access to iOS navigation patterns
- Better memory management

---

### 3. Gesture Handler

**JavaScript Layer (npm):**
```javascript
import {GestureHandlerRootView} from 'react-native-gesture-handler';

// JavaScript gesture API
<GestureHandlerRootView>
  <TouchableOpacity onPress={handlePress}>
    <Text>Press me</Text>
  </TouchableOpacity>
</GestureHandlerRootView>
```

**Native Layer (CocoaPods):**
- Pod: `RNGestureHandler`
- Native iOS touch handling
- Uses iOS `UIGestureRecognizer`
- Direct access to touch events

**Why Native Code Needed:**
- JavaScript event loop is too slow for touch tracking
- Native code processes touches at 120Hz
- Direct hardware access
- No bridge latency for gestures

---

## 📊 Dependency Count Example

From a real React Native project (HabitTracker):

### npm Dependencies: **31 packages**
```json
{
  "react": "19.2.0",
  "react-native": "0.83.1",
  "@react-navigation/native": "^7.1.28",
  "@react-navigation/bottom-tabs": "^7.12.0",
  "@react-native-async-storage/async-storage": "^2.2.0",
  "react-native-gesture-handler": "^2.30.0",
  "react-native-screens": "^4.22.0",
  "react-native-safe-area-context": "^5.5.2"
  // + 23 more packages
}
```

### CocoaPods Dependencies: **83 pods!**

Key pods include:
```ruby
- React-Core              # React Native core bridge
- React-hermes            # JavaScript engine
- RNCAsyncStorage         # AsyncStorage native implementation
- RNGestureHandler        # Gesture handler native code
- RNScreens               # Screen optimization
- react-native-safe-area-context  # Safe area native code
- Yoga                    # Flexbox layout engine (C++)
- boost                   # C++ utility library
- glog                    # Logging library
- fmt                     # Formatting library
- hermes-engine           # JavaScript execution engine
# + 72 more pods
```

**Why so many more pods?**
- React Native core is split into many modules
- Each feature needs native implementation
- C++ dependencies for performance
- iOS SDK integrations
- Build tools and utilities

---

## 🤔 Why Two Package Managers?

### Can't We Just Use npm?

**No, because:**

1. **Different ecosystems**
   - npm: JavaScript ecosystem (Node.js, web, React Native)
   - CocoaPods: iOS ecosystem (Objective-C, Swift)

2. **Different build systems**
   - npm: Metro bundler (JavaScript)
   - CocoaPods: Xcode build system (native compilation)

3. **Performance requirements**
   - JavaScript: Great for UI logic and state
   - Native code: Required for 60fps animations, hardware access

4. **Platform-specific features**
   - iOS APIs can't be accessed from JavaScript directly
   - Need native bridge code

5. **Compilation**
   - JavaScript: Interpreted/JIT compiled
   - Native code: Compiled to machine code

---

## 🔍 What Happens When You Run Commands

### `npm install`

```bash
npm install
```

**Steps:**
1. Reads `package.json`
2. Resolves dependency tree
3. Downloads packages from npm registry (https://registry.npmjs.org)
4. Installs to `node_modules/`
5. Creates/updates `package-lock.json`
6. Runs post-install scripts

**Result:** JavaScript code ready to run in Metro bundler

**Time:** Usually 1-3 minutes

---

### `pod install`

```bash
cd ios
pod install
cd ..
```

**Steps:**
1. Reads `Podfile` (auto-generated by React Native)
2. Analyzes `package.json` to find native modules
3. Resolves dependencies (83+ pods)
4. Downloads pods from CocoaPods specs repo
5. Compiles native code if needed
6. Generates `HabitTracker.xcworkspace`
7. Integrates with Xcode project
8. Links native libraries

**Result:** Native iOS code ready to compile in Xcode

**Time:** Usually 15-30 seconds (after initial setup)

---

## 💡 Key Concepts

### The React Native Bridge

The bridge is what allows JavaScript and native code to communicate:

**JavaScript → Native:**
```javascript
// JavaScript calls native method
AsyncStorage.setItem('key', 'value');

// Behind the scenes:
Bridge.callNative('AsyncStorage', 'setItem', ['key', 'value']);
```

**Native → JavaScript:**
```objc
// Native sends event to JavaScript
[self.bridge enqueueJSCall:@"RCTDeviceEventEmitter"
                    method:@"emit"
                      args:@[@"AppStateChange", @"active"]];
```

**Limitations:**
- Serialization overhead (converts to JSON)
- Asynchronous (can't return values immediately)
- Performance bottleneck for frequent calls

**New Architecture (coming):**
- JSI (JavaScript Interface) - direct access without bridge
- Fabric - new rendering system
- TurboModules - faster native module system

---

## 🎯 Practical Implications

### When Adding a New Feature

**If you want to use a camera:**

1. **Find a library:**
   ```bash
   npm install react-native-camera
   ```

2. **Install native dependencies:**
   ```bash
   cd ios && pod install && cd ..
   ```

3. **Why both?**
   - npm: Installs JavaScript API for using camera
   - pod: Installs native iOS code that accesses `AVFoundation`

### When Updating React Native

1. **Update JavaScript:**
   ```bash
   npm install react-native@latest
   ```

2. **Update native dependencies:**
   ```bash
   cd ios && pod update && pod install && cd ..
   ```

3. **Why both?**
   - New JavaScript features
   - New native implementations
   - Keep versions in sync

---

## 📝 Common Commands Reference

### npm Commands

```bash
# Install all dependencies
npm install

# Add a new package
npm install package-name

# Add dev dependency
npm install --save-dev package-name

# Update packages
npm update

# Check for outdated packages
npm outdated

# Remove a package
npm uninstall package-name

# Clean install
rm -rf node_modules package-lock.json
npm install
```

### CocoaPods Commands

```bash
# Install pods
cd ios && pod install && cd ..

# Update pods
cd ios && pod update && cd ..

# Remove and reinstall
cd ios && pod deintegrate && pod install && cd ..

# Update CocoaPods itself
gem install cocoapods

# Clean pods
cd ios && rm -rf Pods Podfile.lock && pod install && cd ..
```

---

## 🚨 Common Issues

### Issue: npm install works but pod install fails

**Cause:** CocoaPods not installed or Xcode issues

**Solution:**
```bash
# Install CocoaPods
sudo gem install cocoapods

# If Homebrew Ruby:
/opt/homebrew/opt/ruby/bin/gem install cocoapods
```

### Issue: "No such module" error in Xcode

**Cause:** Pods not installed properly

**Solution:**
```bash
cd ios
pod deintegrate
pod install
cd ..
# Make sure to open .xcworkspace not .xcodeproj
```

### Issue: JavaScript works but native features crash

**Cause:** JavaScript and native code out of sync

**Solution:**
```bash
# Clean everything
rm -rf node_modules ios/Pods
npm install
cd ios && pod install && cd ..
```

---

## 🎓 Key Takeaways

1. **npm = JavaScript layer** (your React code, UI logic)
2. **CocoaPods = Native iOS layer** (system integration, performance)
3. **Both are required** for React Native iOS apps
4. They communicate via the **React Native Bridge**
5. You write JavaScript, but native code does the heavy lifting
6. Always run both `npm install` AND `pod install` after cloning
7. When in doubt, clean and reinstall both

---

## 🔗 Analogy: The Car

Think of React Native like a car:

- **npm packages** = Dashboard and controls (what you interact with)
- **CocoaPods** = Engine and mechanical parts (what does the work)
- **React Native Bridge** = Wiring connecting dashboard to engine
- **Your JavaScript code** = Driver pressing buttons and turning wheel
- **iOS SDK** = The road and traffic rules

You press the gas pedal (JavaScript `AsyncStorage.setItem()`), the signal goes through the wiring (Bridge), and the engine responds (Native iOS writes to disk).

---

## 📚 Further Reading

- [React Native Architecture](https://reactnative.dev/docs/architecture-overview)
- [CocoaPods Guides](https://guides.cocoapods.org/)
- [npm Documentation](https://docs.npmjs.com/)
- [React Native New Architecture](https://reactnative.dev/docs/the-new-architecture/landing-page)

---

*Created: February 2026*
*Project Context: HabitTracker app development*
