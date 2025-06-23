# SDL3 DirectDraw Compatibility - Modular Structure

## Overview
The original `sdl3_directdraw_compat.cpp` (1357 lines) has been split into focused, manageable modules to improve maintainability and debugging capabilities, particularly for color/palette issues.

## New Module Structure

### Core Modules

#### 1. **`sdl3_context.hpp/cpp`** - SDL3 System Management
- **Purpose**: SDL3 initialization, window creation, renderer management
- **Key Components**:
  - `SDL3Context` structure
  - `InitializeSDL3Context()` - Set up SDL3 with proper settings
  - `CleanupSDL3Context()` - Clean shutdown
  - `CreateMainWindow_new()` - Window creation hook
  - `CreateCompatibleTexture()` - Helper for texture creation

#### 2. **`surface_management.hpp/cpp`** - DirectDraw Surface Simulation  
- **Purpose**: All DirectDraw interface simulation and surface handling
- **Key Components**:
  - `DummyDirectDrawSurface` structures and vtables
  - Surface locking/unlocking implementations
  - `CreateSDLTextures()` - Creates all game textures
  - `CleanupSDLTextures()` - Proper texture cleanup
  - All surface method implementations (Lock, Unlock, Blt, Flip, etc.)

#### 3. **`palette_system.hpp/cpp`** - Basic Palette Operations
- **Purpose**: Core palette management and SDL3 palette integration
- **Key Components**:
  - `CreateSDL3PaletteSystem()` - Creates indexed surface and palette
  - `UpdateSDL3Palette()` - Updates palette from game data  
  - `CleanupSDL3PaletteSystem()` - Proper cleanup
  - Palette hook installation/uninstallation
  - Basic palette conversion functions

#### 4. **`palette_debug.hpp/cpp`** - Advanced Color Debugging ?
- **Purpose**: Sophisticated palette analysis and color format detection
- **Key Components**:
  - `PaletteAnalysis` structure for format detection
  - `AnalyzePaletteData()` - Automatic RGB/BGR detection
  - `DumpPaletteToFile()` - Creates detailed palette analysis files
  - `UpdateSDL3PaletteWithFormatDetection()` - Smart palette updating
  - Multiple conversion methods for testing different interpretations
  - `ConvertPaletteEntryToRGB()`, `ConvertDirectDrawPaletteToRGB()`, etc.

#### 5. **`sdl3_directdraw_compat_new.cpp`** - Main Integration
- **Purpose**: Puts everything together, main rendering loop
- **Key Components**:
  - `initDirectDraw_new()` - Initialize all systems
  - `ProcessScreenUpdatesAndResources_new()` - Main rendering with debug support
  - `initializeResourceHandlers_new()` - Resource initialization
  - `CleanupSDL3DirectDrawCompat()` - Master cleanup function

## Benefits for Color/Palette Debugging

### ? **Focused Debugging**
- Palette issues are isolated to specific modules
- Easy to test different color format interpretations
- Clear separation between core functionality and debugging tools

### ? **Automatic Analysis** 
- System automatically detects RGB vs BGR vs other formats
- Creates detailed palette dump files (`palette_dump_*.txt`)
- Provides multiple conversion methods to test

### ? **Detailed Diagnostics**
```cpp
// Automatic format detection
PaletteAnalysis analysis;
AnalyzePaletteData(paletteData, &analysis);
if (analysis.dominantColorFormat == 1) {
    // BGR detected - handles color swapping automatically
}
```

### ? **Easy Testing**
- Can quickly switch between RGB/BGR/X68000 interpretations
- Test different conversion methods side by side
- Palette dumps show raw hex, RGB interpretation, BGR interpretation

## Integration Points

### Dependencies (Clean Hierarchy)
```
sdl3_directdraw_compat_new.cpp
?????? surface_management.hpp (surfaces & textures)
?????? palette_system.hpp (basic palette ops)
?????? palette_debug.hpp (advanced debugging)
?????? sdl3_context.hpp (SDL3 system)
```

### Global Variables (Properly Shared)
- All major surfaces (`g_primarySurface`, `g_spriteSurface`, etc.) in `surface_management.cpp`
- All SDL textures (`g_primaryTexture`, `g_spriteTexture`, etc.) in `surface_management.cpp`  
- SDL3 context (`g_sdlContext`) in `sdl3_context.cpp`
- Palette system (`g_sdlPalette`, `g_indexedSurface`) in `palette_system.cpp`

## Usage for Color Issue Resolution

1. **Replace** the original `sdl3_directdraw_compat.cpp` with these modules
2. **Compile** and run - automatically generates `palette_dump_*.txt` files
3. **Analyze** the palette files to see actual color data and format detection
4. **Test** different interpretations by modifying the debug functions
5. **Use** the format detection to automatically handle RGB/BGR inconsistencies

## Files Summary

| File | Lines | Purpose | Status |
|------|-------|---------|---------|
| `sdl3_context.hpp/cpp` | ~300 | SDL3 system management | ? Complete |
| `surface_management.hpp/cpp` | ~450 | DirectDraw simulation | ? Complete |
| `palette_system.hpp/cpp` | ~250 | Basic palette operations | ? Complete |
| `palette_debug.hpp/cpp` | ~400 | Advanced color debugging | ? Complete |
| `sdl3_directdraw_compat_new.cpp` | ~300 | Main integration | ? Complete |

**Total**: ~1700 lines (vs 1357 original) - extra lines provide comprehensive debugging

## What Was Fixed

? **Circular Dependencies** - Removed with proper module hierarchy  
? **Missing Functions** - All original functions accounted for  
? **Global Variable Issues** - Properly shared between modules  
? **Code Duplication** - Eliminated through modular design  
? **Debug Capabilities** - Significantly enhanced for color issues  

This modular approach should finally help identify and resolve the color ordering inconsistencies you described (browns/yellows working vs not working). 