# UI Fixes for SDL3 + ImGui Integration

This document consolidates UI-related fixes for the ImGui overlay system in the SDL3-based game engine.

## Mouse Input Fix for ImGui

### Problem
After pressing F8 to show the ImGui overlay, mouse clicks weren't working on ImGui elements even though the overlay was visible.

### Root Causes
1. **Missing ImGui backend flags** - ImGui wasn't configured for mouse input
2. **SDL3 window focus issues** - The SDL window didn't have proper mouse focus
3. **Event processing gaps** - Mouse events weren't being captured and processed correctly
4. **No focus management** - No system to ensure/restore focus when overlay opens

### Solutions Implemented

#### 1. ImGui Configuration Fix
```cpp
// Added to hooks.cpp - ImGui initialization
io.BackendFlags |= ImGuiBackendFlags_HasMouseCursors;
io.BackendFlags |= ImGuiBackendFlags_HasSetMousePos;
```

#### 2. Enhanced Event Processing
- Added mouse event debugging in SDL event loop
- Added window focus event tracking (mouse enter/leave)
- Improved event processing to ensure mouse events reach ImGui

#### 3. Focus Management System
New functions in `UIUtils`:
- `EnsureWindowFocus()` - Forces SDL window to get proper focus
- `CheckMouseFocus()` - Diagnoses focus state and reports issues
- Automatic focus management when F8 overlay is activated

#### 4. Troubleshooting Tools
Added to Debug Info window:
- **"Check Window Focus"** - Reports current focus state
- **"Force Window Focus"** - Manually fixes focus issues  
- **"Test Mouse Click"** - Shows mouse position and button state

### How to Use Mouse Fix

#### Automatic Fix
The system now automatically ensures proper focus when:
- The overlay is first initialized
- F8 is pressed to show the overlay

#### Manual Troubleshooting
If mouse still doesn't work:
1. Press **F8** to open overlay
2. Open **"Debug Info"** window
3. Click **"Force Window Focus"** button
4. Use **"Test Mouse Click"** to verify mouse is working

#### Debug Information
Check the debug output for messages like:
- `"Mouse focus OK - ImGui should receive mouse input"`
- `"WARNING: Window does not have mouse focus"`
- `"Focus Status: Keyboard=YES, Mouse=YES"`

## ImGui SDL Scaling Fix

### Problem
When SDL renders at a low resolution (e.g., 320x240) but scales up to fullscreen, ImGui receives the fullscreen display size but renders at the low resolution, making all UI elements appear GIGANTIC.

### Solution
The `UIUtils` class now includes automatic scaling compensation for this scenario.

### Quick Fix
To fix the common 320x240 scaling issue:

```cpp
// In your initialization code, call:
UIUtils::SetupCommonSDLScaling();
```

This will automatically detect your display resolution and compensate for 320x240 render scaling.

### Manual Setup
For custom render resolutions:

```cpp
// Example: SDL rendering at 640x480, displaying at 1920x1080
UIUtils::SetupSDLScaling(640.0f, 480.0f, 1920.0f, 1080.0f);

// Or let it auto-detect display resolution:
UIUtils::SetupSDLScaling(640.0f, 480.0f);
```

### Runtime Controls
Press **F8** to open the overlay, then check the **Debug Info** window for:

- **Scaling Controls** section with auto/manual scaling toggles
- **ImGui Scaling (SDL Render Compensation)** section with:
  - Current render and display resolutions
  - Manual resolution adjustment sliders
  - "Auto-Detect Resolutions" button
  - "Apply ImGui Scaling" button
  - **"Fix 320x240 Scaling"** button for the common case

### What It Does
1. **Calculates scaling compensation** - determines how much to shrink ImGui elements
2. **Applies font scaling** - makes text appropriately sized
3. **Scales UI elements** - buttons, windows, etc. are properly sized
4. **Overrides display size** - makes ImGui think it's working with render resolution
5. **Updates game area calculations** - ensures boxes and overlays align correctly

### Technical Details
- `renderResolution`: The actual SDL render target size (e.g., 320x240)
- `displayResolution`: The final display/window size (e.g., 1920x1080)  
- `imguiScaleCompensation`: Calculated ratio to shrink ImGui elements
- Applied every frame in the render loop

## Combined Benefits

### Before Fixes
- **Mouse**: F8 shows overlay but mouse clicks don't work
- **Scaling**: Giant UI elements that don't fit properly

### After Fixes
- **Mouse**: F8 shows overlay with fully functional mouse interaction
- **Scaling**: Properly sized UI that works at any resolution

## Technical Integration

### Mouse Input Technical Details
- Uses both SDL3 focus APIs (`SDL_RaiseWindow`, `SDL_SetWindowFocusable`)
- Falls back to Win32 focus APIs (`SetForegroundWindow`, `SetFocus`) when needed
- Combines SDL event polling with ImGui event processing
- Provides real-time focus state monitoring

### Scaling Technical Details
- `renderResolution`: The actual SDL render target size (e.g., 320x240)
- `displayResolution`: The final display/window size (e.g., 1920x1080)
- `imguiScaleCompensation`: Calculated ratio to shrink ImGui elements
- Applied every frame in the render loop

These fixes ensure reliable mouse input and proper scaling for all ImGui elements in your SDL3-based application, regardless of render resolution vs display resolution setup.