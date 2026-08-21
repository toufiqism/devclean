# devclean

Interactive disk cleanup for macOS developer machines.

Dev tooling quietly eats hundreds of gigabytes — stale iOS simulator runtimes,
Android NDK copies, Gradle caches, emulator snapshots, browser caches. `devclean`
finds them, shows what each one costs, and deletes only what you tick.

Python 3 standard library only. No dependencies, no install step, one file.

```
  ████████████████░░░░░░░░░░░░ 130.6 GB used / 228.3 GB  97.6 GB free (57%)
  selected: 6 item(s)  reclaims 7.3 GB  -> 104.9 GB free
  --------------------------------------------------------------------------
  Xcode & Simulators
  [x]    3.0 GB  SAFE   Simulator dyld cache
  [ ]  412.2 MB  REVIEW Xcode CodingAssistant
  Android
  [x]    3.9 GB  SAFE   Emulator snapshots
          Cold-boot state; emulators still work without them
  [ ]    2.3 GB  REVIEW Android SDK system-images
  [x]    2.2 MB  SAFE   Gradle native
    [kept: NDK pinned by your projects: 28.2.13676358]
    [MISSING: projects pin NDK 27.0.12077973 but it is not installed]
  Browser
  [x]   12.4 MB  SAFE   Chrome caches
  Caches
  [x]  335.8 MB  SAFE   ~/Library/Caches (app caches)
  --------------------------------------------------------------------------
  ↑↓ move   space toggle   a all-safe   n none   v invert   enter clean   q quit
```

## Install

```sh
git clone https://github.com/<you>/devclean.git
install -m 755 devclean/devclean ~/bin/devclean   # anywhere on your PATH
```

If `~/bin` isn't on your PATH yet, add it to `~/.zprofile` (or `~/.zshrc`):

```sh
export PATH="$HOME/bin:$PATH"
```

## Usage

```sh
devclean              # interactive
devclean --dry-run    # show the plan, change nothing
devclean --safe-only  # only offer SAFE items
```

| Key     | Action                          |
| ------- | ------------------------------- |
| `↑` `↓` | move                            |
| `space` | toggle the highlighted item     |
| `a`     | select everything marked SAFE   |
| `n`     | select nothing                  |
| `v`     | invert selection                |
| `enter` | review the plan, then confirm   |
| `q`     | quit without changing anything  |

Piping the output (`devclean | less`) prints a plain report and exits without
touching anything, which makes it safe to run from a script or a cron job.

## Risk tiers

| Tier     | Meaning                                                                    |
| -------- | -------------------------------------------------------------------------- |
| `SAFE`   | Pure cache. Regenerates on its own. Preselected.                            |
| `REVIEW` | Real cost to lose — a re-download, or state you may want. Never preselected.|
| `MOVE`   | Not a cache. Relocated to `~/Desktop/devclean-quarantine/`, never deleted.   |

Nothing is removed until you review the plan and type `yes`.

## What it looks at

**Xcode & simulators** — DerivedData, iOS DeviceSupport, Archives, simulator
devices, installed runtimes, the dyld cache.

**Android** — NDK versions, Gradle caches, SDK system-images and sources,
emulator snapshots, superseded Android Studio / JetBrains config directories.

**Browsers** — Chrome, Chrome Beta, Brave, Edge, Vivaldi, Firefox. Cache
subdirectories only.

**Caches** — `~/.cache`, `~/Library/Caches`, npm, Yarn, pnpm, pip, uv, Homebrew,
CocoaPods, Carthage, Go, Cargo, Puppeteer, Electron, Playwright.

**Junk** — JVM/Node crash dumps, diagnostic reports, simulator logs, Trash.

## Safety

The interesting part of a cleanup tool is what it refuses to touch.

- **Hard denylist.** Documents, Desktop, Development, Projects, `~/src`, `.ssh`,
  `.gnupg`, Keychains, Mail, Messages, iCloud Drive. Every path is checked by
  `guard()` immediately before removal, not just when the list was built.
- **Rooted deletes.** A path must resolve inside `$HOME` or `/Library/Developer`.
  Anything else is skipped even if a probe somehow produced it.
- **Browser profiles are never deleted** — only known cache subdirectories
  *inside* them. Passwords, bookmarks, history, cookies, extensions and
  IndexedDB are unreachable by design. The browser must be quit first.
- **Newest iOS runtime is kept.** Older ones are offered individually.
- **Pinned NDKs are kept**, resolved two ways: an explicit `ndkVersion = "x.y.z"`
  in your Gradle files, and `ndkVersion = flutter.ndkVersion`, which is looked up
  in the Flutter SDK's `gradle_utils.dart`. Vendored SDK trees are excluded —
  Flutter's own integration tests pin versions for Flutter's CI that no local app
  builds against, and counting those hides the ones that matter. If a project
  pins an NDK that *isn't* installed, that's reported too.
- **Active IDE config is kept.** Among several Android Studio / JetBrains
  version directories, the most recently modified one is never offered.
- **Large non-cache files are moved, not deleted.** Anything over 200 MB found
  inside browser storage (`File System`, `IndexedDB`, `blob_storage`) is
  quarantined to the Desktop so you can look at it first. This exists because a
  3 GB file sitting in Chrome's sandbox turned out to be a two-hour video, not
  garbage.
- **Permission errors degrade quietly.** Paths needing Full Disk Access are
  skipped and reported as a `sudo` command you can run yourself; they never
  abort the scan.

## Requirements

macOS, Python 3.9+. Both ship with the system or with Xcode Command Line Tools.

## License

MIT
