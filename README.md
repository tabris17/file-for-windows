# file Command and libmagic for Windows

Dependent projects:

- file: <https://github.com/file/file>
- pcre2: <https://github.com/PCRE2Project/pcre2>
- zlib: <https://github.com/madler/zlib>
- bzip2: <https://sourceware.org/bzip2/>
- liblzma: <https://tukaani.org/xz/>
- zstd: <https://github.com/facebook/zstd>
- lzlib: <https://www.nongnu.org/lzip/lzlib.html>

Requirements:

- [PowerShell](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows)
- [WinLibs](https://winlibs.com/)
- [CMake](https://cmake.org/) (also included in WinLibs)
- [Ninja](https://ninja-build.org/) (also included in WinLibs)

To build with MinGW, use the following commands:

```powershell
$env:CC=(Get-Command gcc).Source
$env:CXX=(Get-Command g++).Source
cmake -GNinja -Bbuild -S.
ninja -C build magic magic_file file
```

or

```powershell
cmake -D CMAKE_C_COMPILER=$( (Get-Command gcc).Source ) -D CMAKE_CXX_COMPILER=$( (Get-Command g++).Source ) -GNinja -Bbuild -S.
cmake --build build --config Release --target magic magic_file file
```
