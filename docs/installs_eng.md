# Environment Setup and Compilation

**Project:** A/B Compare
**Reference Date:** August 21, 2026
**Status:** Builds validated on Windows 10 and Windows 11 x64; release ZIP validated on Windows 10; Linux x86_64 AppImage built on Ubuntu 22.04 LTS and validated on KDE/Wayland.

## 1. Scope of This Document

This document specifies the commands required to produce the executable from the repository root.

The compilation commands assume the project adheres to the following contract:

- CMake target: `ab-compare`;
- Linux executable: `ab-compare`;
- Windows executable: `ab-compare.exe`;
- macOS bundle: `AB Compare.app`;
- Tests registered via CTest;
- CMake `install()` rules that place the application in `bin/` (on Linux/Windows) or as a bundle (on macOS);
- Qt dependencies (as per MVP Express) declared within the repository;
- libebur128 1.2.6, downloaded with a fixed SHA-256 hash and statically linked (no additional shared libraries);
- No scripts dependent on any specific IDE.

The Windows commands below utilize Qt 6.9.3 and Visual Studio 2022. They apply to Windows 10 and Windows 11 x64. The Arch Linux commands have been validated for native builds; the AppImage production process is separate.

## 2. Validation Status by System

| System | Intended Build | Native Tests | Release Format |
|---|---:|---:|---:|
| Arch Linux x64, KDE Plasma, Wayland, PipeWire | Validated from source and via AppImage | Automated tests and Wayland/XWayland smoke tests | AppImage |
| Ubuntu 22.04 LTS x64 | reproducible AppImage build in a container | automated with virtual PulseAudio output | AppImage; interactive GNOME test (to be completed) |
| Windows 10 x64 | validated | validated | portable ZIP |
| Windows 11 x64 | validated | validated | portable ZIP |
| macOS Intel/Apple Silicon | prepared | currently unavailable | deferred |

There are no plans to compile for Windows or macOS from Arch Linux. Each version must be built natively on its respective OS or on a CI runner using the same OS.

## 3. Arch Linux — KDE Plasma, Konsole, and fish

### 3.1 Installing tools and libraries

In Konsole using fish:

```fish
sudo pacman -Syu
sudo pacman -S --needed base-devel cmake ninja git pkgconf qt6-base qt6-declarative qt6-multimedia qt6-multimedia-ffmpeg qt6-wayland
```

The Qt packages come from the official Arch repositories; no AUR packages are required. A network connection is necessary during the initial CMake configuration to download the official libebur128 1.2.6 archive. CMake verifies its SHA-256 checksum before extraction and compiles it statically.

Verification:

```fish
cmake --version
ninja --version
qtpaths6 --qt-version
```

### 3.2 Configuring a Release build

From the root of the repository:

```fish
cmake -S . -B build/linux-release -G Ninja -DCMAKE_BUILD_TYPE=Release
```

### 3.3 Build

```fish
cmake --build build/linux-release --parallel (nproc)
```

The expected development executable is:

```text
build/linux-release/app/ab-compare
```

If the final CMake directory structure places the target in a different subdirectory, this path will need to be updated accordingly; the name `ab-compare` must remain as specified.

### 3.4 Run tests

```fish
ctest --test-dir build/linux-release --output-on-failure
```

### 3.5 Create a local installable directory structure

```fish
cmake --install build/linux-release --prefix dist/linux
```

The expected installed executable is:

```text
dist/linux/bin/ab-compare
```

Run this version:

```fish
./dist/linux/bin/ab-compare
```

This directory structure still relies on the system's Qt and audio libraries. Refer to the next section to produce a distributable AppImage package.

### 3.6 Rebuild after modifications

```fish
cmake --build build/linux-release --parallel (nproc)
ctest --test-dir build/linux-release --output-on-failure
```

There is no need to re-run the configuration command unless the CMake files, dependencies, or options have changed.

### 3.7 Produce the Linux x86_64 AppImage

Docker must be installed and accessible to the current user. From the root of the repository:

```fish
./scripts/linux/build-appimage-container.sh
```

The script builds the application within a fixed Ubuntu 22.04 LTS image, installs Qt 6.9.3, runs CTest using WAV, FLAC, MP3, and Opus tracks, and bundles the Qt Multimedia, XCB, and Wayland plugins. The downloaded `linuxdeploy` and `linuxdeploy-plugin-qt` tools are verified via SHA-256 checksums.
The build process also verifies that the Qt Wayland-EGL client integration and the `xdgdesktopportal` theme are indeed present in the final AppImage. On Linux, the application prefers using the desktop portal to access the system file picker, provided that a D-Bus session is available and the portal service is responsive. This logic is embedded within the binary itself—and thus applies to `build/linux-release/app/ab-compare` as well, not just the AppImage. The probe is limited to a one-second timeout. The application respects any `QT_QPA_PLATFORMTHEME` variable defined by the user, and it retains the Qt Quick dialog as a fallback should the probe, the service, or the necessary querying tool be unavailable.

The generated files are:

```text
dist/release/AudioABComparator-1.0.0-linux-x86_64.AppImage
dist/release/SHA256SUMS-linux
```

Local validation:

```fish
chmod +x dist/release/AudioABComparator-1.0.0-linux-x86_64.AppImage
./dist/release/AudioABComparator-1.0.0-linux-x86_64.AppImage --smoke-test
./dist/release/AudioABComparator-1.0.0-linux-x86_64.AppImage
env QT_QPA_PLATFORM=xcb ./dist/release/AudioABComparator-1.0.0-linux-x86_64.AppImage
```

The last two commands should open and keep the window visible—the first running natively under Wayland, and the second under XWayland/XCB. These checks are essential, as the smoke test alone terminates before the rendering of the first frame is initialized.

A detailed report can be found in [`validation_linux.md`](validation_linux.md).
## 4. Windows 10 and Windows 11 x64 — PowerShell

### 4.1 Install general tools

Open PowerShell as an administrator to perform the installations:

```powershell
winget install --id Git.Git -e --source winget
winget install --id Kitware.CMake -e --source winget
winget install --id Ninja-build.Ninja -e --source winget
winget install --id Python.Python.3.12 -e --source winget
winget install --id 7zip.7zip -e --source winget
```

Close and reopen PowerShell to refresh the `PATH`.

### 4.2 Install the MSVC compiler

In an administrator PowerShell session:

```powershell
winget install --id Microsoft.VisualStudio.2022.BuildTools -e --source winget --override "--wait --passive --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended"
```

This workload installs the C++ compiler, the CMake/MSBuild tools required for Visual Studio, and a supported Windows SDK. 
### 4.3 Installing Qt 6.9.3

`aqtinstall` allows for a reproducible installation of Qt via the command line:

```powershell
py -3.12 -m ensurepip --upgrade
py -3.12 -m pip install --user --upgrade aqtinstall
py -3.12 -m aqt install-qt windows desktop 6.9.3 win64_msvc2022_64 -O C:\Qt --modules qtmultimedia
```

Expected Qt installation path:

```text
C:\Qt\6.9.3\msvc2022_64
```

Verification:

```powershell
& 'C:\Qt\6.9.3\msvc2022_64\bin\qmake.exe' -query QT_VERSION
```

The Qt version is intentionally fixed to ensure reproducible builds. Any updates to this version must be explicitly applied both in this document and within the CI pipeline.

### 4.4 Analysis Dependency

There is no need to install vcpkg, libsndfile, or a libebur128 DLL. During the initial configuration, CMake downloads the official libebur128 1.2.6 archive, verifies it against the SHA-256 hash specified in `CMakeLists.txt`, and then links it statically. Consequently, the build machine must have HTTPS access to GitHub during this step; subsequent reconfigurations will reuse the source files already present in the build directory. ### 4.5 Loading the MSVC environment

In a new, non-administrator PowerShell window:

```powershell
& "${env:ProgramFiles(x86)}\Microsoft Visual Studio\2022\BuildTools\Common7\Tools\Launch-VsDevShell.ps1" -Arch amd64 -HostArch amd64
cl.exe
```

`cl.exe` should display its version. A message indicating that no source files were provided is normal.

### 4.6 Configuring a Release build

From the root of the repository, in the same PowerShell window:

```powershell
cmake -S . -B build\windows-release -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=C:\Qt\6.9.3\msvc2022_64
```

### 4.7 Building and testing

```powershell
cmake --build build\windows-release --parallel
$env:PATH = 'C:\Qt\6.9.3\msvc2022_64\bin;' + $env:PATH
ctest --test-dir build\windows-release --output-on-failure
```

The real-format regression test (`validatesExternalReleaseFormats`) runs only when `AB_COMPARE_TEST_FORMAT_DIR` points to a folder that contains `track-a` and `track-b` in WAV, FLAC, MP3 and Opus. With FFmpeg available, the fixtures can be generated as follows:

```powershell
$fixtures = 'C:\Dev\audioab-format-fixtures'
New-Item -ItemType Directory -Path $fixtures -Force | Out-Null
foreach ($track in @(@('a', 440), @('b', 660))) {
    foreach ($extension in 'wav', 'flac', 'mp3', 'opus') {
        ffmpeg -hide_banner -loglevel error -y -f lavfi -i "sine=frequency=$($track[1]):duration=8" -ar 48000 -ac 2 "$fixtures\track-$($track[0]).$extension"
    }
}
$env:AB_COMPARE_TEST_FORMAT_DIR = $fixtures
ctest --test-dir build\windows-release --output-on-failure
```

Expected development executable:

```text
build\windows-release\ab-compare.exe
```

### 4.8 Creating the distributable folder

```powershell
cmake --install build\windows-release --prefix dist\windows
& 'C:\Qt\6.9.3\msvc2022_64\bin\windeployqt.exe' --release --qmldir app\ui dist\windows\bin\ab-compare.exe
```

Expected installed executable:

```text
dist\windows\bin\ab-compare.exe
```

Running this version:

```powershell
& '.\dist\windows\bin\ab-compare.exe'
```

libebur128 is already statically linked into the executable. `windeployqt` deploys Qt, its plugins, and the FFmpeg backend provided by Qt; no libebur128 DLLs should appear. Verification on a clean Windows machine remains mandatory before release.

### 4.9 Creating the release ZIP

After a successful build and testing:

```powershell
& '.\scripts\windows\package-release.ps1' -BuildDirectory 'build\windows-release' -QtRoot 'C:\Qt\6.9.3\msvc2022_64'
Get-Content '.\dist\release\SHA256SUMS'
```

The script creates `AudioABComparator-1.0.0-windows-x86_64.zip` and `SHA256SUMS`. The ZIP contains the executable at its root, the necessary DLLs/plugins/QML files, `README.txt`, `LICENSE`, `THIRD_PARTY_NOTICES.md`, the LGPL texts, and the MIT license for libebur128. It is intended solely for GitHub Releases attachments and must not be committed to Git.

## 5. macOS — procedure prepared but not validated

This section does not constitute official support. It should be executed on an Intel-based and/or Apple Silicon Mac, or within a macOS CI environment, and subsequently refined based on the actual results.

### 5.1 Install Xcode and Homebrew

In Terminal (using zsh):

```zsh
xcode-select --install
```

Then, install Homebrew from its official website if `brew` is not already available.

### 5.2 Install Qt and build tools

```zsh
brew update
brew install cmake ninja qt git
```

### 5.3 Analysis dependency

As with Linux and Windows, CMake downloads, verifies, and statically compiles libebur128 1.2.6. No additional audio dependency managers are required.

### 5.4 Configure, build, and test on Apple Silicon

From the root of the repository:

```zsh
cmake -S . -B build/macos-release -G Ninja \
-DCMAKE_BUILD_TYPE=Release \
-DCMAKE_PREFIX_PATH="$(brew --prefix qt)"
cmake --build build/macos-release --parallel "$(sysctl -n hw.logicalcpu)"
ctest --test-dir build/macos-release --output-on-failure
cmake --install build/macos-release --prefix dist/macos
```

Expected bundle:

```text
dist/macos/AB Compare.app
```

### 5.5 Deploy Qt into the bundle

```zsh
"$(brew --prefix qt)/bin/macdeployqt" "dist/macos/AB Compare.app" -dmg
```

This command does not sign or notarize the application. Signing, notarization, audio permissions, actual paths for vcpkg libraries, and operation on both Intel and Apple Silicon architectures remain to be validated for macOS.

## 6. Minimum verification after each build

On each validated OS:

1. launch the executable from its distribution folder;
2. load two stereo WAV files;
3. create a selection of at least 5 seconds;
4. play, pause, stop, and enable looping;
5. toggle between A and B using the keyboard at least 100 times;
6. verify the Up/Down controls and the counters;
7. modify the keyboard shortcuts, then restart the application;
8. perform an analysis on the entire file and on the selection, then confirm that any conversion is detailed separately for A and B;
9. verify that the keyboard shortcuts persist, while files, selections, votes, and analyses are cleared;
10. ensure there are no recurring clicks, drift, or audio underruns;
11. run the application on a clean machine before release.

## 7. Sources for procedures

- [Arch Linux Qt Multimedia package](https://archlinux.org/packages/extra/x86_64/qt6-multimedia/)
- [Official libebur128 1.2.6 source](https://github.com/jiixyj/libebur128/releases/tag/v1.2.6)
- [Installing Qt with aqtinstall](https://aqtinstall.readthedocs.io/en/stable/getting_started.html)
- [Visual Studio installation parameters](https://learn.microsoft.com/visualstudio/install/use-command-line-parameters-to-install-visual-studio)
- [Homebrew Qt formula](https://formulae.brew.sh/formula/qt.html)