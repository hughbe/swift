# Getting Started with Swift on Windows

## MSVC
- Windows doesn't currently have a build script. You'll need to run commands manually to build Swift on Windows.
- Release/RelWithDebInfo modes have not been tested and may not be supported.
- x86 has not been tested.
- Windows support for Swift is very much a WIP, and may not work on your system.

### 1. Install dependencies
- Make sure to add Python, CMake and Ninja to your `Path` environment variable
1. Latest version (2.7.12 tested) of [Python 2](https://www.python.org/downloads/)
2. Latest version (3.7.0-rc3 tested) of [CMake](https://cmake.org/download/)
3. Latest version (1.7.1 tested) of [Ninja](https://github.com/ninja-build/ninja/releases/latest)
4. Latest version (2015 Update 3 tested) of [Visual Studio](https://www.visualstudio.com/downloads/)
- Make sure to include `Programming Languages|Visual C++`, and `Windows and Web Development|Universal Windows App Development|Windows SDK` in your installation.
- Windows SDK 10.0.10240 was tested. Some later versions (e.g. 10.0.14393) are known not to work, as they are not supported by `compiler-rt`.

### 2. Clone the repositories
1. Create a folder to contain all the Swift repositories
2. `apple/swift-cmark` into a folder named `cmark`
3. `apple/swift-clang` into a folder named `clang`
5. `apple/swift-llvm` into a folder named `llvm`
5. `apple/swift` into a folder named `swift`
- Currently, other repositories in the Swift project have not been tested, and may not be supported.

### 3. Build ICU
1. Download and extract the [ICU source code](http://site.icu-project.org/download) to a folder named `icu` in the same directory as the other Swift project repositories.
2. Open `src/win32/allinone.sln` in Visual Studio.
3. Make sure to select the correct architecture from the drop-down in Visual Studio.
4. Right click on the solution in the Solution Explorer window and select `Build Solution`.
5. When this is done, add the `<icu-source>/bin` folder to your `Path` environment variable.

### 4. Get ready
- From within a **developer** command prompt, execute the following command if you have an x64 PC.
```
"C:/Program Files (x86)/Microsoft Visual Studio 14.0/VC/vcvarsall.bat" amd64
```
- Then adapt the following command, and run it.
```
set swift_source_dir=path-to-directory-containing-all-cloned-repositories
```

### 5. Build CMark
- This must be done from within a developer command prompt, and could take up to 10 minutes.
```
mkdir "%swift_source_dir%/build/Ninja-DebugAssert/cmark-windows-amd64"
pushd "%swift_source_dir%/build/Ninja-DebugAssert/cmark-windows-amd64"
cmake -G "Ninja" "%swift_source_dir%/cmark"
popd
cmake --build "%swift_source_dir%/build/Ninja-DebugAssert/cmark-windows-amd64/"
```

### 6. Build LLVM/Clang/Compiler-RT
- This must be done from within a developer command prompt, and could take up to 5 hours.
```
mklink /J "%swift_source_dir%/llvm/tools/clang" "%swift_source_dir%my-swift/clang"
mklink /J "%swift_source_dir%/llvm/tools/compiler-rt" "%swift_source_dir%/compiler-rt"
mkdir "%swift_source_dir%/build/Ninja-DebugAssert/llvm-windows-amd64"
pushd "%swift_source_dir%/build/Ninja-DebugAssert/llvm-windows-amd64"
cmake -G "Ninja"^
 -DLLVM_ENABLE_ASSERTIONS=TRUE^
 -DCMAKE_BUILD_TYPE=Debug^
 -DLLVM_TOOL_SWIFT_BUILD=NO^
 -DLLVM_INCLUDE_DOCS=TRUE^
 -DLLVM_TOOL_COMPILER_RT_BUILD=TRUE^
 -DLLVM_BUILD_EXTERNAL_COMPILER_RT=TRUE^
 -DLLVM_LIT_ARGS=-sv^
 "%swift_source_dir%/llvm"
popd
cmake --build "%swift_source_dir%/build/Ninja-DebugAssert/llvm-windows-amd64"
```

### 7. Build Swift
- This must be done from within a developer command prompt, and could take up to 2 hours.
- You may need to adjust the SWIFT_WINDOWS_LIB_DIRECTORY parameter depending on your target platform or Windows SDK version.
```
mkdir "%swift_source_dir%/build/Ninja-DebugAssert/swift-windows-amd64/ninja"
pushd "%swift_source_dir%/build/Ninja-DebugAssert/swift-windows-amd64/ninja"
cmake -G "Ninja" "%swift_source_dir%/swift"^
 -DCMAKE_CXX_FLAGS="/std:c++14"^
 -DCMAKE_BUILD_TYPE=Debug^
 -DSWIFT_PATH_TO_CMARK_SOURCE="%swift_source_dir%/cmark"^
 -DSWIFT_PATH_TO_CMARK_BUILD="%swift_source_dir%/build/Ninja-DebugAssert/cmark-windows-amd64"^
 -DSWIFT_CMARK_LIBRARY_DIR="%swift_source_dir%/build/Ninja-DebugAssert/cmark-windows-amd64/src"^
 -DSWIFT_PATH_TO_LLVM_SOURCE="%swift_source_dir%/llvm"^
 -DSWIFT_PATH_TO_LLVM_BUILD="%swift_source_dir%/build/Ninja-DebugAssert/llvm-windows-amd64"^
 -DSWIFT_PATH_TO_CLANG_SOURCE="%swift_source_dir%/llvm/tools/clang"^
 -DSWIFT_PATH_TO_CLANG_BUILD="%swift_source_dir%/build/Ninja-DebugAssert/llvm-windows-amd64"^
 -DSWIFT_WINDOWS_ICU="%swift_source_dir%/icu"^
 -DSWIFT_INCLUDE_DOCS=FALSE^
 -DSWIFT_INCLUDE_TESTS=FALSE^
 -DSWIFT_BUILD_SDK_OVERLAY=FALSE^
 -DSWIFT_BUILD_RUNTIME_WITH_HOST_COMPILER=TRUE^
 -DSWIFT_WINDOWS_LIB_DIRECTORY="C:/Program Files (x86)/Windows Kits/10/Lib/10.0.10240.0/ucrt/x64"
popd
cmake --build "%swift_source_dir%/build/Ninja-DebugAssert/swift-windows-amd64/ninja"
```


## Windows Subsystem for Linux (WSL)
- Note that all compiled Swift binaries are only executable within Bash on Windows and are Ubuntu, not Windows, executables.
- Make sure to run all commands from Bash, or the project won't compile.

###  1. Install WSL
Install and run the latest version of [Bash on Ubuntu on Windows](https://msdn.microsoft.com/en-gb/commandline/wsl/about) installed on your PC.
```
bash
```

### 2. Install dependencies
Install the developer dependencies needed to compile the Swift project. These are identical to the Ubuntu dependencies, with the addition of `make`.
```bash
sudo apt-get install git make cmake ninja-build clang python uuid-dev libicu-dev icu-devtools libbsd-dev libedit-dev libxml2-dev libsqlite3-dev swig libpython-dev libncurses5-dev pkg-config libblocksruntime-dev libcurl4-openssl-dev
```

### 3. Upgrade clang
Install a version of clang with C++ 14 support - the default version of clang on WSL results in linker errors during compilation.
```bash
sudo apt-get install clang-3.6
sudo update-alternatives --install /usr/bin/clang clang /usr/bin/clang-3.6 100
sudo update-alternatives --install /usr/bin/clang++ clang++ /usr/bin/clang++-3.6 100
```

### 4. Upgrade CMake
Install the latest version of CMake - Swift uses new CMake features such as `IN_LIST` and won't build without these features.
```bash
wget http://www.cmake.org/files/v3.5/cmake-3.6.2.tar.gz
tar xf cmake-3.6.2.tar.gz
cd cmake-3.6.2
./configure
make
sudo make install
sudo update-alternatives --install /usr/bin/cmake cmake /usr/local/bin/cmake 1 --force
cmake --version # This should print 3.6.2
```

### 6. Clone and build the Swift project
```bash
mkdir swift-source
cd swift-source
git clone https://github.com/apple/swift.git
./swift/utils/update-checkout --clone
./swift/utils/build-script -r
```
### 7. Hello, Windows (Subsystem for Linux)
```bash
cd ./build/Ninja-RelWithDebInfoAssert/swift-linux-x86_64/bin # This path may depend on your build configuration
echo 'print("Hello, Windows")' >> test.swift
swiftc test.swift
./test # Hello, Windows
```
