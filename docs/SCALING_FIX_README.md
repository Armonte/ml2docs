# ImGui SDL Scaling Fix

## Problem
When SDL renders at a low resolution (e.g., 320x240) but scales up to fullscreen, ImGui receives the fullscreen display size but renders at the low resolution, making all UI elements appear GIGANTIC.

## Solution
The `UIUtils` class now includes automatic scaling compensation for this scenario.

## Quick Fix
To fix the common 320x240 scaling issue:

```cpp
// In your initialization code, call:
UIUtils::SetupCommonSDLScaling();
```

This will automatically detect your display resolution and compensate for 320x240 render scaling.

## Manual Setup
For custom render resolutions:

```cpp
// Example: SDL rendering at 640x480, displaying at 1920x1080
UIUtils::SetupSDLScaling(640.0f, 480.0f, 1920.0f, 1080.0f);

// Or let it auto-detect display resolution:
UIUtils::SetupSDLScaling(640.0f, 480.0f);
```

## Runtime Controls
Press **F8** to open the overlay, then check the **Debug Info** window for:

- **Scaling Controls** section with auto/manual scaling toggles
- **ImGui Scaling (SDL Render Compensation)** section with:
  - Current render and display resolutions
  - Manual resolution adjustment sliders
  - "Auto-Detect Resolutions" button
  - "Apply ImGui Scaling" button
  - **"Fix 320x240 Scaling"** button for the common case

## What It Does
1. **Calculates scaling compensation** - determines how much to shrink ImGui elements
2. **Applies font scaling** - makes text appropriately sized
3. **Scales UI elements** - buttons, windows, etc. are properly sized
4. **Overrides display size** - makes ImGui think it's working with render resolution
5. **Updates game area calculations** - ensures boxes and overlays align correctly

## Technical Details
- `renderResolution`: The actual SDL render target size (e.g., 320x240)
- `displayResolution`: The final display/window size (e.g., 1920x1080)  
- `imguiScaleCompensation`: Calculated ratio to shrink ImGui elements
- Applied every frame in the render loop

## Before/After
- **Before**: Giant UI elements that don't fit properly
- **After**: Properly sized UI that works at any resolution

This fix ensures ImGui scales correctly regardless of your SDL render resolution vs display resolution setup. 