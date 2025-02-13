# file Command and libmagic for Windows

Dependent projects:

- file: <https://github.com/file/file>
- pcre2: <https://github.com/PCRE2Project/pcre2>

To build with MinGW, use the following commands:

```powershell
$env:CC=(Get-Command gcc).Source
$env:CXX=(Get-Command g++).Source
cmake -D CMAKE_GENERATOR=Ninja .
```

or

```powershell
cmake -D CMAKE_C_COMPILER=$( (Get-Command gcc).Source ) -D CMAKE_CXX_COMPILER=$( (Get-Command g++).Source ) -D CMAKE_GENERATOR=Ninja .
```