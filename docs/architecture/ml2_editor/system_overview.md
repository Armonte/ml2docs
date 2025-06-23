# Moon Lights 2 Editor - Complete Documentation Summary

## Overview

This document provides a complete summary of all documentation created for the Moon Lights 2 all-in-one editing tool using Clay UI. This serves as the master reference for implementing the editor.

## ? **Documentation Files**

1. **`ML2_FILE_FORMATS_TECHNICAL_NOTES.md`** - Core technical file format specifications
2. **`ML2_CLAY_UI_INTEGRATION_DESIGN.md`** - Clay UI integration and implementation design
3. **`ML2_TIMELINE_SYSTEM_DESIGN.md`** - Timeline and state machine editor design
4. **`ML2_IMPLEMENTATION_ROADMAP.md`** - Implementation phases and roadmap

## ? **Key Systems Documented**

### **File Format Systems**
- **Character System**: 14 characters, file naming conventions (`HI.SP`, `HI.PAL`, etc.)
- **Palettes (.PAL)**: RGB555 format, 256 colors, 16 palettes of 16 colors each
- **Sprites (.SP)**: 4-bit indexed color, 8x16 tile storage, multiple 256x256 pages
- **Frame Data (.FRM)**: Big-endian format, pattern-based sprite positioning
- **VSE Data (.VSE)**: 48-byte move structure, 256 entries, frame-based timing

### **Memory Structures**
- **Player Structure**: 284 bytes (0x11C), complete combat and state data
- **Entity System**: 8 slots per player, 284 bytes each, projectiles and effects
- **Memory Addresses**: All exact game memory locations documented

### **VSE System Integration**
- **Dual Data Systems**: Pattern data (sprites/hitboxes) + Movement data (velocity) with separate offsets
- **VSE Table Structure**: 256 entries ~ 8 bytes, bytes 2-3 = pattern offset, bytes 6-7 = movement offset  
- **Duration System**: Frame-based timing (1-255 frames at 60 FPS)
- **Movement Algorithm**: Direct file access with control flow (jumps, end markers)
- **Hitbox Types**: Primary/secondary attack, hurtbox, collision, clashbox
- **Player-VSE Mapping**: Direct field mapping between VSE moves and player structure
- **Entity Spawning**: VSE moves can spawn entities via `unk_actionEntityRelated`

## ? **Clay UI Editor Design**

### **Professional Layout**
```
„¡„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„¢
„  File Edit View Tools Help                    [Menu Bar]      „ 
„¥„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„¦„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„¦„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„§
„  Chars   „  [Sprite] [Hitbox] [VSE] [Anim] [Pal] „  Properties  „ 
„  & Files „                                      „  Panel       „ 
„          „          Main Workspace              „              „ 
„  [List]  „          (Tabbed Interface)          „  [Details]   „ 
„          „                                      „              „ 
„¥„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„¨„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„¨„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„§
„  Timeline Panel                                              „ 
„  [Play] [Stop] [<<<] [>>>] ????????????????????              „ 
„  Sprite Track   ????????????????????????????????????????     „ 
„  Hitbox Track   ????????????????????????????????????????     „ 
„  Audio Track    ????????????????????????????????????????     „ 
„¤„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„£
```

### **Custom Elements**
- **ML2_ELEMENT_SPRITE_VIEWER**: Sprite rendering with zoom/pan
- **ML2_ELEMENT_HITBOX_OVERLAY**: Hitbox visualization over sprites
- **ML2_ELEMENT_TIMELINE_TRACK**: Multi-track timeline with VSE frames
- **ML2_ELEMENT_PALETTE_STRIP**: Color palette display and editing
- **ML2_ELEMENT_VSE_FRAME_DATA**: VSE frame property editor

### **Tab System**
1. **Sprite Viewer**: Browse and view character sprites with zoom/pan
2. **Hitbox Editor**: Edit hitboxes overlaid on sprites
3. **VSE Editor**: Frame-by-frame VSE editing with dual pattern/movement data
4. **Movement Editor**: Dedicated velocity and position editing with visual preview
5. **Animation Preview**: Real-time animation playback with movement simulation
6. **Palette Editor**: Color palette editing and effects
7. **State Machine**: Visual state transition editor

## ?? **Timeline System**

### **Multi-Track Timeline**
- **Sprite Track**: VSE pattern data with duration visualization
- **Movement Track**: Velocity data with position preview (NEW)
- **Hitbox Track**: Attack/hurt/collision box visualization
- **Audio Track**: Sound effect cues and timing
- **Control Flow Track**: Animation jumps and loop visualization (NEW)
- **Playback Controls**: Play, pause, stop, step frame-by-frame
- **Zoom & Scrub**: Professional timeline navigation
- **Dual Data Sync**: Pattern and movement tracks stay synchronized

### **Frame-Accurate Editing**
- **Duration Control**: Edit frame duration (1-255 frames)
- **Timeline Scrubbing**: Click to seek, real-time preview
- **Frame Interpolation**: Smooth transitions between frames
- **Loop Points**: Set animation loop start/end points

## ? **Fighting Game Features**

### **Complete Hitbox System**
```c
// Box types supported
enum BoxType {
    Collision,    // Character body collision
    Hitbox1,      // Primary attack box
    Hitbox2,      // Secondary attack box
    Hurtbox1,     // Primary vulnerable area
    Hurtbox2,     // Secondary vulnerable area
    Clashbox,     // Attack collision detection
    Entity        // Projectile/entity boxes
};
```

### **State Machine Editor**
- **Visual Graph**: Node-based state transition editing
- **Condition Editor**: Input and frame-based transition conditions
- **State Properties**: Health, meter, timing requirements
- **Branching Logic**: Complex state flow with multiple paths

### **Combat Data Integration**
```c
// VSE move properties directly editable - PATTERN DATA (48 bytes)
struct VSEMove {
    uint8_t sprite;           // FRM pattern index
    int8_t duration;          // Frame duration (1-255)
    int8_t damageDealt;       // Attack damage
    int8_t stunDealt;         // Hitstun frames
    int8_t blockType;         // High/low/overhead
    int8_t hitType;           // Hit properties
    int8_t sfx_1, sfx_2;      // Sound effects
    // ... complete hitbox data
};

// VSE movement properties - MOVEMENT DATA (2 bytes per frame)
struct VSEMovementFrame {
    int8_t x_movement;        // X velocity (-128 to +127)
    int8_t y_movement;        // Y velocity (-128 to +127)
};
```

### **Critical VSE Architecture Discovery**
- **Two Separate Systems**: Pattern data and movement data are stored in different file locations
- **Separate Offset Pointers**: VSE table entries have two different offset fields pointing to each system
- **Direct File Access Required**: Movement data must be accessed using calculated file offsets
- **Control Flow Integration**: Movement data includes jump commands and end markers
- **Horizontal Flipping**: Movement data is flipped using two's complement for left-facing characters

## ? **Technical Implementation**

### **Data Flow**
1. **File Loading**: Load character data (.SP, .PAL, .FRM, .VSE)
2. **Dual VSE Parsing**: 
   - Parse VSE table (256 entries ~ 8 bytes)
   - Load pattern data (48 bytes per move)  
   - Keep raw file data for movement access (2 bytes per frame)
3. **Structure Parsing**: Convert file data to editor structures
4. **UI Rendering**: Clay UI renders custom elements with dual data sources
5. **Real-time Preview**: Changes in pattern/movement data immediately reflected across tabs
6. **Export**: Save modifications to both pattern and movement data systems

### **Memory Integration**
```c
// Direct game memory compatibility
#define PLAYER1_ADDRESS      0x006B2BC0    // Player 1 structure
#define PLAYER2_ADDRESS      0x006B2CDC    // Player 2 structure
#define P1_ENTITIES_ADDRESS  0x00688590    // Player 1 entities (8 slots)
#define P2_ENTITIES_ADDRESS  0x00688E70    // Player 2 entities (8 slots)
#define VSE_BASE_ADDRESS     0x004DAAB0    // VSE data in memory
```

### **Performance Optimization**
- **Lazy Loading**: Load sprites only when needed
- **Texture Caching**: Cache converted textures
- **Viewport Culling**: Render only visible elements
- **Frame Limiting**: 60 FPS with frame pacing

## ? **Implementation Phases**

### **Phase 1: Foundation (Weeks 1-2)**
- Basic Clay UI integration with SDL3
- File loading system (sprites, palettes, FRM, VSE)
- Basic tabbed interface
- Simple sprite viewer

### **Phase 2: Core Features (Weeks 3-4)**
- Complete hitbox visualization
- VSE frame editing
- Timeline basic functionality
- Palette editing

### **Phase 3: Advanced Features (Weeks 5-6)**
- Timeline multi-track system
- Animation playback engine
- State machine visual editor
- Real-time preview integration

### **Phase 4: Polish & Features (Weeks 7-8)**
- Professional shortcuts and workflows
- Export/import functionality
- Undo/redo system
- Performance optimization

## ? **Key Technical Details**

### **File Format Compatibility**
- **No Format Changes**: All existing files remain compatible
- **Exact Byte Layout**: Maintain precise structure layouts
- **Endianness Handling**: Proper big/little endian conversion
- **Memory Address Preservation**: Game memory layout unchanged

### **Editor Features**
- **Professional UI**: Modern IDE-like interface
- **Cross-Tool Integration**: Changes in one tool affect others
- **Frame-Accurate Timing**: Exact 60 FPS frame timing
- **Real-Time Preview**: Immediate visual feedback

### **Fighting Game Specifics**
- **Multiple Hitbox Types**: Complete fighting game hitbox system
- **Frame Data Editing**: Frame-by-frame animation control
- **Dual Data Editing**: Separate UI for pattern data vs movement data
- **Movement Preview**: Real-time character position preview with velocity
- **Control Flow Visualization**: Visual representation of animation jumps and loops
- **State Machine Logic**: Visual state transition editing
- **Combat Properties**: Damage, stun, block type, hit properties

## ? **Final Result**

A **professional fighting game animation editor** that rivals commercial tools like:
- **Guilty Gear Strive's animation editor**
- **Street Fighter development tools**
- **Arc System Works animation pipeline**

With features specifically tailored for Moon Lights 2's unique dual VSE system and fighting game mechanics.

### **Unique Technical Achievements**
- **Dual Data System Mastery**: First editor to properly handle ML2's separate pattern/movement data architecture
- **Frame-Perfect Movement**: Real-time movement simulation with exact game physics
- **Control Flow Visualization**: Visual editing of animation loops, jumps, and state transitions
- **Cross-System Synchronization**: Pattern and movement data stay perfectly synchronized
- **Professional Timeline**: Multi-track editing rivaling video editing software
- **Fighting Game Precision**: Frame-accurate editing essential for competitive play

### **Technical Complexity Handled**
- **File Format Expertise**: Complete understanding of 4 different binary formats (.SP, .PAL, .FRM, .VSE)
- **Memory Layout Precision**: Exact compatibility with game memory structures
- **Dual Offset Calculation**: Correct handling of separate pattern/movement offset systems
- **Real-time Preview**: Immediate visual feedback for both sprite and movement changes
- **Performance Optimization**: Efficient handling of large character data sets

## ? **Documentation Structure**

```
ML2_Editor_Documentation/
„¥„Ÿ„Ÿ ML2_FILE_FORMATS_TECHNICAL_NOTES.md      # Core file format specs
„¥„Ÿ„Ÿ ML2_CLAY_UI_INTEGRATION_DESIGN.md        # UI implementation design
„¥„Ÿ„Ÿ ML2_TIMELINE_SYSTEM_DESIGN.md            # Timeline & state machine
„¥„Ÿ„Ÿ ML2_IMPLEMENTATION_ROADMAP.md            # Development phases
„¤„Ÿ„Ÿ ML2_EDITOR_COMPLETE_DOCUMENTATION_SUMMARY.md  # This summary
```

This documentation provides everything needed to implement a professional-grade fighting game animation editor for Moon Lights 2! ??