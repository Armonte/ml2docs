# Build Instructions for ML2 Fix

## Important: 32-bit Compilation Required

This project **must** be compiled as 32-bit because it hooks into a 32-bit game. A 64-bit DLL cannot be injected into a 32-bit process.

## Prerequisites

### Install MSYS2 MINGW32 Environment

1. Install MSYS2 from https://www.msys2.org/
2. Open **MSYS2 MINGW32** terminal (not MSYS2 UCRT64 or regular MSYS2)
3. Install required packages:
```bash
pacman -S mingw-w64-i686-gcc
pacman -S mingw-w64-i686-cmake
pacman -S mingw-w64-i686-make
pacman -S mingw-w64-i686-pkg-config
```

## Build Methods

### Method 1: Windows Batch Script (Recommended)
```cmd
build_mingw32.bat
```
This automatically launches the MINGW32 environment and builds the project.

### Method 2: From MINGW32 Terminal
1. Open **MSYS2 MINGW32** terminal
2. Navigate to project: `cd /c/Users/stjra/source/repos/mlfixtest`
3. Test environment: `./test_mingw32_env`
4. Build: `./make_build_mingw32 && ./go`

### Method 3: Original Scripts (Updated)
From MINGW32 terminal:
```bash
./make_build && ./go
```

## Troubleshooting

### Environment Check
Run `./test_mingw32_env` to verify your environment is correctly set up.

### Common Issues
1. **"compiler not found"** - Make sure you're in MINGW32 terminal, not UCRT64
2. **Assembly errors** - These should be resolved with 32-bit compilation
3. **Wrong architecture** - Verify `gcc -dumpmachine` shows `i686-w64-mingw32`

### Required Environment Variables
- `MSYSTEM=MINGW32`
- `MINGW_PREFIX=/mingw32`

## Output Files
- `hook.dll` - 32-bit DLL for injection into the game
- `launcher.exe` - 32-bit launcher executable

## Migration from Previous Build System

The build system has been updated from Cygwin/older MinGW to MSYS2 MINGW32:
- Better CMake integration
- More reliable 32-bit compilation
- Proper dependency management
- Cleaner build process

## Why 32-bit?

This project hooks into game memory using techniques that require matching architecture:
- DLL injection requires same bitness (32-bit DLL Å® 32-bit process)
- Memory addresses are 32-bit specific
- Assembly hooks are written for x86 (32-bit) instruction set

Attempting to use 64-bit compilation will result in:
- Injection failures
- Assembly compilation errors
- Runtime incompatibilities 