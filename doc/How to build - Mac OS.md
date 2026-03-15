
# Building Bambu Studio on MacOS

## Environment setup

Install Following tools:

- Xcode from app store
- Cmake
- git
- gettext
- ninja (recommended, for faster builds)

Cmake, git, gettext, and ninja can be installed from brew:
```shell
brew install cmake git gettext ninja
```

## Building with BuildMac.sh (recommended)

The easiest way to build is using the provided `BuildMac.sh` script.

### First-time build

This builds both the dependencies and the slicer:

```shell
./BuildMac.sh -a arm64 -x
```

For x86_64 Mac:
```shell
./BuildMac.sh -a x86_64 -x
```

### Development workflow after `git pull`

You do **not** need to do a full rebuild every time you pull changes or edit source files.
CMake tracks which files have changed and only recompiles what is necessary.

**After `git pull` when only source files changed (most common case):**
```shell
./BuildMac.sh -s -a arm64 -x
```
The `-s` flag skips rebuilding the dependencies (which rarely change) and only rebuilds
the slicer. CMake automatically detects changed `.cpp`/`.h` files and does an incremental
compile — unchanged files are not recompiled.

**Fastest incremental rebuild (skip CMake reconfiguration entirely):**
```shell
./BuildMac.sh -s -a arm64 -x -b
```
The `-b` flag skips CMake's configuration step as well. Use this when you have only edited
source files and have **not** changed `CMakeLists.txt`, added/removed files, or updated
build configuration. This is the fastest option for a tight edit–compile loop.

**After `git pull` when `deps/` or `CMakeLists.txt` changed:**
```shell
./BuildMac.sh -a arm64 -x
```
Rebuild both deps and the slicer when the dependencies or build configuration have changed.

### Build script options

Run `./BuildMac.sh -h` for the full list of options:

```
Usage: ./BuildMac.sh [-1][-d][-s][-x][-b][-c][-a][-t][-p]
   -d: Build deps
   -a: Set ARCHITECTURE (arm64 or x86_64 or universal)
   -s: Build slicer only (skip rebuilding deps)
   -x: Use Ninja CMake generator (faster than default Xcode generator)
   -b: Build without reconfiguring CMake (incremental build, fastest option)
   -c: Set CMake build configuration, default is Release
   -1: limit builds to 1 core (where possible)
```

---

## Manual build (advanced)

The sections below describe how to build manually using CMake directly, which gives
more control but requires managing paths yourself.

### Building the dependencies manually

You need to build the dependencies of BambuStudio first. (Only needs for the first time)

Suppose you download the codes into `/Users/_username_/work/projects/BambuStudio`.

Create a directory to store the built dependencies: `/Users/_username_/work/projects/BambuStudio_dep`.
**(Please make sure to replace the username with the one on your computer)**

Then:

```shell
cd BambuStudio/deps
mkdir build
cd build
```

Next, for arm64 architecture:
```shell
cmake ../ -DDESTDIR="/Users/username/work/projects/BambuStudio_dep" -DOPENSSL_ARCH="darwin64-arm64-cc"
make
```

Or, for x86 architeccture:
```shell
cmake ../ -DDESTDIR="/Users/username/work/projects/BambuStudio_dep" -DOPENSSL_ARCH="darwin64-x86_64-cc"
make -jN
```
(N can be a number between 1 and the max cpu number)  

### Building Bambu Studio manually

Create a directory to store the installed files at `/Users/username/work/projects/BambuStudio/install_dir`:

```shell
cd BambuStudio
mkdir install_dir
mkdir build
cd build
```

To build using CMake:

```shell
cmake ..  -DBBL_RELEASE_TO_PUBLIC=1 -DCMAKE_PREFIX_PATH="/Users/username/work/projects/BambuStudio_dep/usr/local" -DCMAKE_INSTALL_PREFIX="../install_dir" -DCMAKE_BUILD_TYPE=Release -DCMAKE_MACOSX_RPATH=ON -DCMAKE_INSTALL_RPATH="/Users/username/work/projects/BambuStudio_dep/usr/local" -DCMAKE_MACOSX_BUNDLE=on
cmake --build . --target install --config Release -jN
```

For subsequent builds after changing source files, you only need to re-run the build
step — CMake will incrementally recompile only changed files:

```shell
cd build
cmake --build . --target install --config Release -jN
```

To build for use with XCode:

```shell
cmake .. -GXcode -DBBL_RELEASE_TO_PUBLIC=1 -DCMAKE_PREFIX_PATH="/Users/username/work/projects/BambuStudio_dep/usr/local" -DCMAKE_INSTALL_PREFIX="../install_dir" -DCMAKE_BUILD_TYPE=Release -DCMAKE_MACOSX_RPATH=ON -DCMAKE_INSTALL_RPATH="/Users/username/work/projects/BambuStudio_dep/usr/local" -DCMAKE_MACOSX_BUNDLE=on
```
