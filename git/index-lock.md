# Git index.lock — How Git Protects Its Staging Area

Understanding the mutex mechanism Git uses to prevent concurrent index corruption.

## Quick Summary

| Aspect | Details |
|--------|---------|
| What | `.git/index.lock` — a file-based mutex for the Git staging area |
| Why | Prevents two Git processes from writing to the index simultaneously |
| Created by | Any command that modifies the index (`git add`, `git commit`, `git merge`, etc.) |
| Removed by | The same command, upon completion or failure |
| Problem case | Process crash leaves an orphaned lock file, blocking all subsequent Git commands |
| Fix | `rm .git/index.lock` |

---

## Context

When two Git processes try to modify the index at the same time (e.g., running `git add` in a terminal while an editor or CI tool is also staging changes), the staging area can become corrupted. Git needs a way to ensure only one process writes to the index at any given moment.

This is especially relevant when working with AI coding agents like Claude Code or IDE Git integrations, where multiple tools may run Git commands concurrently against the same repository.

---

## Explanation

### Creation — Atomic File Lock

When a Git command needs to modify the index, it creates `.git/index.lock` using the `open()` system call with `O_CREAT | O_EXCL` flags. This is an atomic operation at the filesystem level — it means "create this file, but **fail immediately** if it already exists." Only one process can ever win this race.

```c
// Simplified from Git source (lockfile.c)
fd = open(path, O_RDWR | O_CREAT | O_EXCL, 0666);
```

If another process already holds the lock, Git prints the familiar error and exits:

```
fatal: Unable to create '/path/to/.git/index.lock': File exists.
Another git process seems to be running in this repository...
```

### Write Phase — Write-Ahead to Lock File

While holding the lock, Git writes the **new** index state to `index.lock`, not directly to `index`. The original `index` file remains untouched during the entire operation. This is a write-ahead pattern that protects against corruption.

### Normal Completion — Atomic Rename

On success, Git renames `index.lock` to `index`:

```c
rename(lock_path, index_path);
```

`rename()` is atomic on POSIX filesystems — the index is never in a half-written state. The lock file disappears as a side effect because it **became** the new index.

### Failure — Cleanup

If the operation fails (e.g., a merge conflict, user abort), Git deletes `index.lock` and leaves the original `index` untouched. The staging area is exactly as it was before the operation started.

### Stale Lock — Orphaned File

If the process is killed (`kill -9`) or crashes before cleanup, `index.lock` is left behind with no owning process. Every subsequent Git command sees the lock file and refuses to proceed, even though no process is actually holding it.

### Lifecycle Diagram

```
                  git add / commit / merge
                          |
                          v
               +---------------------+
               |  open() with        |
               |  O_CREAT | O_EXCL   |
               +------+------+-------+
                      |      |
              success |      | file exists
                      v      v
             +----------+  ERROR: "Another git
             |  Write    |  process seems to be
             |  new      |  running..."
             |  index    |
             |  state    |
             +--+---+----+
                |   |
        success |   | failure/crash
                |   |
       +--------+   +--------+
       v                     v
  +----------+        +-----------+
  | rename   |        | Process   |
  | lock ->  |        | died?     |
  | index    |        +--+----+---+
  +----------+           |    |
       |           cleanup|    |no cleanup
       v                 v    v
  Lock gone        Lock       STALE LOCK
  (became index)   deleted    (manual rm needed)
```

---

## Real-World Example

While working with Claude Code and a terminal simultaneously, both tried to run Git commands at the same time. Claude Code was running `git add` to stage files while a manual `git commit` was initiated in the terminal. The result:

```
fatal: Unable to create '/path/to/project/.git/index.lock': File exists.
Another git process seems to be running in this repository...
```

The fix was straightforward — verify no Git process was running, then remove the stale lock:

```bash
ps aux | grep git
rm .git/index.lock
git status
```

---

## How to Fix a Stale Lock

### 1. Verify no Git process is actually running

```bash
ps aux | grep git
```

### 2. Remove the stale lock

```bash
rm .git/index.lock
```

### 3. Verify the index is intact

```bash
git status
```

If `git status` works normally, everything is fine. The original `index` was never modified because the write-ahead pattern protected it.

---

## Key Takeaways

1. `index.lock` is a filesystem-level mutex using `O_CREAT | O_EXCL` for atomic creation — only one process can win the lock.
2. Git uses a **write-ahead pattern**: new state is written to `index.lock`, then atomically renamed to `index`. The staging area is never half-written.
3. On success, the lock file **becomes** the new index via `rename()`. On failure, it's deleted. Either way, it disappears.
4. A stale lock means a process died without cleaning up. It's always safe to `rm .git/index.lock` as long as no Git process is actually running.
5. Working with AI coding agents or IDEs that run Git commands introduces concurrency — be aware of the potential for lock contention.

---

## Common Pitfalls

- **Deleting `index.lock` while a Git process is still running** — Always check `ps aux | grep git` first. Removing the lock from under a running process can corrupt the index.
- **Running concurrent Git operations** — Two terminals, an IDE + CLI, or Claude Code + terminal can all race for the lock. Wait for one to finish before starting another.
- **Force-killing Git processes** — `kill -9` prevents cleanup. Use `kill` (SIGTERM) first to give Git a chance to remove its lock file.
- **Forgetting `pod install` or similar post-lock cleanup** — After resolving a stale lock during a merge or rebase, remember to complete the interrupted operation rather than just removing the lock and moving on.

---

## Prevention Tips

When working with tools that also run Git commands (like Claude Code, IDE Git integrations, or Git hooks):

- **Avoid concurrent Git operations.** Wait for one process to finish before starting another.
- **Use separate worktrees** if you need parallel Git operations on the same repo: `git worktree add ../my-worktree feature-branch`.
- **Check `git status`** before running commands if another tool was recently modifying the repo.

---

## Further Reading

- [Git Internals - The Index](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)
- [Git source: lockfile.c](https://github.com/git/git/blob/master/lockfile.c)
- [Git source: read-cache.c](https://github.com/git/git/blob/master/read-cache.c) (index read/write logic)

---

*Created: February 2026*
*Project Context: Concurrent Git operations between Claude Code and terminal*
