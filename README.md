# devclean

Interactive disk cleanup for developer machines. Runs on macOS, Linux and Windows.

Dev tooling quietly eats hundreds of gigabytes — stale iOS simulator runtimes,
Android NDK copies, Gradle caches, emulator snapshots, browser caches. `devclean`
finds them, shows what each one costs, and deletes only what you tick.

Python 3 standard library only. No dependencies, no install step, one file.

```
  █████████████████████░░░░░░░ 384.2 GB used / 500.0 GB  115.8 GB free (77%)
  selected: 5 item(s)  reclaims 23.8 GB  -> 139.6 GB free
  --------------------------------------------------------------------------
  Xcode & Simulators
  [x]   11.4 GB  SAFE   Simulator dyld cache
  [ ]    8.9 GB  REVIEW Simulator runtime iOS 18.4
  [x]    6.2 GB  SAFE   iOS DeviceSupport
  Android
  [ ]    3.1 GB  REVIEW NDK 25.2.9519653
  [ ]    2.8 GB  REVIEW Gradle caches
  [x]    1.9 GB  SAFE   Emulator snapshots
    [kept: NDK pinned by your projects: 27.2.12479018]
  Browser
  [ ]    1.4 GB  MOVE   Stranded file in Chrome File System
          Matroska/WebM video - moved out, not deleted
  [x]  742.0 MB  SAFE   Chrome caches
  Caches
  [x]    3.6 GB  SAFE   ~/.cache (app caches)
  --------------------------------------------------------------------------
  ↑↓ move   space toggle   a all-safe   n none   v invert   enter clean   q quit
```

## Install

`devclean` is a single file with no dependencies, so installing it means putting
that one file somewhere on your `PATH`.

**Read it before you run it.** It deletes things, and that isn't something to
take on trust from a stranger's README. Every recipe below downloads the file
first and runs it second, which leaves you a moment to look:

```sh
less ~/bin/devclean        # or: notepad %LOCALAPPDATA%\Programs\devclean\devclean.py
```

### macOS

Python 3 arrives with the Xcode Command Line Tools; run `xcode-select --install`
if you don't have them yet.

```sh
mkdir -p ~/bin
curl -fsSL -o ~/bin/devclean \
  https://raw.githubusercontent.com/toufiqism/devclean/main/devclean
chmod +x ~/bin/devclean

# zsh is the default shell on macOS
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zprofile
exec zsh
```

Verify:

```sh
command -v devclean && devclean --dry-run
```

### Linux

Most distributions ship Python 3. `~/.local/bin` is the XDG convention and is
already on `PATH` on Debian, Ubuntu, Fedora and Arch, so the PATH line is
usually unnecessary — check with `echo $PATH` before adding it.

```sh
mkdir -p ~/.local/bin
curl -fsSL -o ~/.local/bin/devclean \
  https://raw.githubusercontent.com/toufiqism/devclean/main/devclean
chmod +x ~/.local/bin/devclean

# only if ~/.local/bin is not already on PATH
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
exec bash
```

Verify:

```sh
command -v devclean && devclean --dry-run
```

### Windows

Install Python 3 from [python.org](https://www.python.org/downloads/) or the
Microsoft Store first, then in **PowerShell**:

```powershell
$dir = "$env:LOCALAPPDATA\Programs\devclean"
New-Item -ItemType Directory -Force -Path $dir | Out-Null

curl.exe -fsSL -o "$dir\devclean.py" `
  https://raw.githubusercontent.com/toufiqism/devclean/main/devclean

# Windows has no shebang, so a tiny shim makes `devclean` work as a command
Set-Content "$dir\devclean.cmd" '@py "%~dp0devclean.py" %*'

# add to PATH for future sessions
$user = [Environment]::GetEnvironmentVariable('Path', 'User')
[Environment]::SetEnvironmentVariable('Path', "$user;$dir", 'User')
$env:Path += ";$dir"
```

Verify:

```powershell
devclean --dry-run
```

The shim calls `py`, the launcher the python.org installer sets up. If your
install doesn't have it, swap `py` for `python` in that `Set-Content` line.

Use **Windows Terminal** or PowerShell 7 if you can. The tool asks the console
to enable ANSI colour and falls back to plain text when that's refused, but the
legacy `cmd.exe` window also can't render the box-drawing characters in the
disk bar, so it substitutes ASCII.

### From a clone

Preferable if you want the history, or intend to pull updates:

```sh
git clone https://github.com/toufiqism/devclean.git
cd devclean
install -m 755 devclean ~/bin/devclean       # macOS
install -m 755 devclean ~/.local/bin/devclean # Linux
```

### First run

`--dry-run` scans and prints the plan without touching anything. It's the right
way to see what the tool would do on your machine before letting it do it:

```sh
devclean --dry-run
```

### Updating

From a clone:

```sh
cd devclean && git pull && install -m 755 devclean ~/bin/devclean
```

Standalone — rerun the same `curl` from your platform's section; it overwrites
in place.

#### Windows

Rerun only the download, not the whole install block. The `devclean.cmd` shim
and the `PATH` entry are already in place, and running the block again would
add the folder to `PATH` a second time. In PowerShell:

```powershell
curl.exe -fsSL -o "$env:LOCALAPPDATA\Programs\devclean\devclean.py" `
  https://raw.githubusercontent.com/toufiqism/devclean/main/devclean
```

If you keep a clone instead, pull and copy the script over the installed one:

```powershell
cd devclean
git pull
Copy-Item devclean "$env:LOCALAPPDATA\Programs\devclean\devclean.py"
```

Then run `devclean --dry-run` to check the new version starts.

### Uninstalling

```sh
rm ~/bin/devclean            # macOS
rm ~/.local/bin/devclean     # Linux
```

```powershell
Remove-Item -Recurse "$env:LOCALAPPDATA\Programs\devclean"
```

It writes no config and keeps no state, so nothing else is left behind — except
`Desktop/devclean-quarantine/`, if it ever moved a file there for you.

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
| `MOVE`   | Not a cache. Relocated to `Desktop/devclean-quarantine/`, never deleted.    |

Nothing is removed until you review the plan and type `yes`.

## What it looks at

**Xcode & simulators** — DerivedData, iOS DeviceSupport, Archives, simulator
devices, installed runtimes, the dyld cache.

**Android** — NDK versions, Gradle caches, SDK system-images and sources,
emulator snapshots, superseded Android Studio / JetBrains config directories.

**Browsers** — Chrome, Chrome Beta, Brave, Edge, Vivaldi, Firefox. Cache
subdirectories only.

**Caches** — the platform cache root, plus npm, Yarn, pip, uv, Go, Cargo, Maven,
pub, Puppeteer, Electron, Playwright, and per-platform extras (Homebrew,
CocoaPods and Carthage on macOS; Flatpak and thumbnails on Linux; NuGet and
`%LOCALAPPDATA%\Temp` on Windows).

**Stremio** — the streaming server's torrent cache (`stremio-cache`), at its
default location for each platform and the Flatpak build, or wherever you moved
it in Stremio's settings. Only the cache's contents go; `server-settings.json`
and the app are left alone. Stremio must be quit first.

**Junk** — JVM/Node crash dumps, diagnostic reports, simulator logs, Trash.

## Safety

The interesting part of a cleanup tool is what it refuses to touch.

- **Hard denylist.** Documents, Desktop, Development, Projects, `~/src`, `.ssh`,
  `.gnupg`, Keychains, Mail, Messages, iCloud Drive. Every path is checked by
  `guard()` immediately before removal, not just when the list was built.
- **Rooted deletes.** A path must resolve inside `$HOME`, or one of the
  platform roots (`/Library/Developer` on macOS; a relocated `XDG_CACHE_HOME`
  or `%LOCALAPPDATA%`). Anything else is skipped even if a probe produced it.
  The one addition is a Stremio cache moved to another drive: the `stremio-cache`
  directory itself is allowed, never the drive or folder it was moved into.
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
  quarantined to `Desktop/devclean-quarantine/` — or the home directory, on
  systems without a Desktop — so you can look at it first. This exists because a
  3 GB file sitting in Chrome's sandbox turned out to be a two-hour video, not
  garbage.
- **Permission errors degrade quietly.** Paths needing elevation are skipped
  and reported back as a `sudo` (or `Remove-Item`) command you can run
  yourself; they never abort the scan.

## Requirements

Python 3.9+ and nothing else — no third-party packages.

| Platform | Notes |
| -------- | ----- |
| macOS    | Python ships with the Xcode Command Line Tools. Full feature set, including Xcode and iOS simulator probes. |
| Linux    | Uses the XDG base directories, honouring `XDG_CACHE_HOME` and friends. |
| Windows  | Uses `%LOCALAPPDATA%` and `%APPDATA%`. Requires Windows 10 1511+ for coloured output; it degrades to plain text on older consoles. |

Probes that don't apply to the host are skipped, so each platform only ever
sees targets that exist on it. Xcode and the iOS simulator sections appear on
macOS alone; the Recycle Bin is deliberately left alone on Windows, because it
is not safe to manipulate as plain files.

One deliberate asymmetry: macOS and Linux get a generic sweep of their cache
root (`~/Library/Caches`, `~/.cache`), because both operating systems define
one. Windows does not — `%LOCALAPPDATA%` mixes real application state in with
caches — so on Windows only specifically named caches are ever offered.

## License

MIT
