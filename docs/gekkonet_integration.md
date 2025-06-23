# GekkoNet Integration for Moon Lights 2

## Current Progress

### Completed
- Created basic GekkoNet integration structure
- Set up input state structure for Moon Lights 2
- Modified battle loop to initialize and process GekkoNet updates
- Identified key integration points between systems

### In Progress
- Input system mapping
- State synchronization
- Timing integration
- Cleanup procedures

## Technical Details

### Input Structure
```cpp
struct ML2Input {
    union {
        struct {
            char up : 1;
            char down : 1;
            char left : 1;
            char right : 1;
            char A : 1;
            char B : 1;
            char C : 1;

        } buttons;
        unsigned char value;
    } input;
};
```

### GekkoNet Configuration
```cpp
GekkoConfig g_config = {
    .num_players = 2,
    .input_size = sizeof(ML2Input),
    .max_spectators = 0,
    .input_prediction_window = 0
};
```

## Next Steps

### 1. Input System Mapping
- [ ] Analyze Moon Lights 2's current input system
- [ ] Create mapping between ML2Input and game's input state
- [ ] Implement input state capture in battle loop
- [ ] Test input synchronization

### 2. State Synchronization
- [ ] Identify key game state variables
- [ ] Create state save/load functions
- [ ] Implement rollback handling
- [ ] Test state synchronization

### 3. Timing Integration
- [ ] Verify compatibility with ProcessGameFrameHook
- [ ] Implement frame pacing
- [ ] Test timing synchronization
- [ ] Add delay simulation if needed

### 4. Cleanup and Error Handling
- [ ] Implement proper session cleanup
- [ ] Add error handling for network issues
- [ ] Create recovery mechanisms
- [ ] Test error scenarios

## Known Issues
1. Include path errors for chrono and GekkoNet headers
2. Need to verify input bitfield matches game's input system
3. Timing system compatibility needs testing
4. State synchronization strategy needs refinement

## Dependencies
- Windows.h
- GekkoNet library
- C++ standard library (chrono, utility)

## Files
- `gekkonet_integration.cpp` - Core integration code
- `gekkonet_integration.h` - Header file (needs to be created)
- `battleloop.cpp` - Modified battle loop with GekkoNet integration

## Notes
- Current timing system in ProcessGameFrameHook should work with GekkoNet
- Need to ensure proper cleanup of GekkoNet session on game exit
- Consider adding debug logging for network state
- May need to adjust input prediction window based on testing 