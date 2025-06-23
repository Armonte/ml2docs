# Mouse Input Fix for ImGui

## Problem
After pressing F8 to show the ImGui overlay, mouse clicks weren't working on ImGui elements even though the overlay was visible.

## Root Causes
1. **Missing ImGui backend flags** - ImGui wasn't configured for mouse input
2. **SDL3 window focus issues** - The SDL window didn't have proper mouse focus
3. **Event processing gaps** - Mouse events weren't being captured and processed correctly
4. **No focus management** - No system to ensure/restore focus when overlay opens

## Solutions Implemented

### 1. ImGui Configuration Fix
```cpp
// Added to hooks.cpp - ImGui initialization
io.BackendFlags |= ImGuiBackendFlags_HasMouseCursors;
io.BackendFlags |= ImGuiBackendFlags_HasSetMousePos;
```

### 2. Enhanced Event Processing
- Added mouse event debugging in SDL event loop
- Added window focus event tracking (mouse enter/leave)
- Improved event processing to ensure mouse events reach ImGui

### 3. Focus Management System
New functions in `UIUtils`:
- `EnsureWindowFocus()` - Forces SDL window to get proper focus
- `CheckMouseFocus()` - Diagnoses focus state and reports issues
- Automatic focus management when F8 overlay is activated

### 4. Troubleshooting Tools
Added to Debug Info window:
- **"Check Window Focus"** - Reports current focus state
- **"Force Window Focus"** - Manually fixes focus issues  
- **"Test Mouse Click"** - Shows mouse position and button state

## How to Use

### Automatic Fix
The system now automatically ensures proper focus when:
- The overlay is first initialized
- F8 is pressed to show the overlay

### Manual Troubleshooting
If mouse still doesn't work:
1. Press **F8** to open overlay
2. Open **"Debug Info"** window
3. Click **"Force Window Focus"** button
4. Use **"Test Mouse Click"** to verify mouse is working

### Debug Information
Check the debug output for messages like:
- `"Mouse focus OK - ImGui should receive mouse input"`
- `"WARNING: Window does not have mouse focus"`
- `"Focus Status: Keyboard=YES, Mouse=YES"`

## Technical Details
- Uses both SDL3 focus APIs (`SDL_RaiseWindow`, `SDL_SetWindowFocusable`)
- Falls back to Win32 focus APIs (`SetForegroundWindow`, `SetFocus`) when needed
- Combines SDL event polling with ImGui event processing
- Provides real-time focus state monitoring

## Before/After
- **Before**: F8 shows overlay but mouse clicks don't work
- **After**: F8 shows overlay with fully functional mouse interaction

This fix ensures reliable mouse input for all ImGui elements in your SDL3-based application. 