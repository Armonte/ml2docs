# Moon Lights 2 File Formats & Systems Technical Notes

## Overview

This document provides comprehensive technical notes on the ML2 file formats and systems based on the existing codebase. These notes serve as the foundation for the Clay UI editor implementation.

## Character System

### Character Database
```c
// From character_info.h/cpp
struct CharacterInfo {
    const char* name;        // Human-readable name
    int id;                  // Unique character ID  
    const char* filePrefix;  // 2-letter file prefix (e.g., "HI", "UM")
};

// 14 total characters supported
const CharacterInfo characters[14] = {
    {"Hikaru", 0, "HI"},    {"Umi", 1, "UM"},       {"Fuu", 2, "FU"},
    {"Rei", 4, "RE"},       {"Asuka", 5, "AS"},     {"Ryoryo", 6, "RR"},
    {"Ryoko", 7, "RY"},     {"Aeka", 8, "AE"},      {"Moon", 9, "US"},
    {"Chibi", 10, "CH"},    {"Hotaru", 11, "HO"},   {"Harumi", 12, "HA"},
    {"Kaworu", 13, "KA"},   {"Multi", 14, "MA"}
};
```

### File Naming Convention
- **Sprites**: `{PREFIX}.SP` (e.g., `HI.SP` for Hikaru)
- **Palettes**: `{PREFIX}.PAL` (e.g., `HI.PAL`)
- **Frame Data**: `{PREFIX}.FRM` (e.g., `HI.FRM`)
- **VSE Data**: `{PREFIX}.VSE` (e.g., `HI.VSE`)

## Palette System (.PAL Files)

### File Format Specification
```c
// Palette file structure
struct PaletteFile {
    uint16_t colors[256];    // 256 colors, 2 bytes each
};

// Color format: 16-bit RGB555 (little-endian)
// Bit layout: [0bGGGGGBBBBBRRRRR]
```

### Color Conversion Algorithm
```c
// From palette_system.cpp - LoadPaletteFile()
for (int p = 0; p < 256; ++p) {
    uint8_t byte1, byte2;
    file.read(&byte1, 1);  // Low byte
    file.read(&byte2, 1);  // High byte
    
    int Thisp = (byte1 * 256) + byte2;  // Combine bytes
    
    // Extract RGB components (5-bit each)
    uint8_t r = (Thisp >> 3) & 0xF8;    // Red: bits 0-4, shifted to 8-bit
    uint8_t g = (Thisp >> 8) & 0xF8;    // Green: bits 5-9, shifted to 8-bit  
    uint8_t b = (Thisp << 2) & 0xF8;    // Blue: bits 10-14, shifted to 8-bit
    
    // Alpha channel handling
    float alpha = (p % 16 == 0) ? 0.0f : 1.0f;  // First color of each 16-color palette is transparent
    
    colors[p] = ImVec4(r/255.0f, g/255.0f, b/255.0f, alpha);
}
```

### Palette Organization
- **256 total colors** arranged as **16 palettes of 16 colors each**
- **Palette 0-15**: Each palette represents different character states, special effects, etc.
- **Color 0 of each palette**: Always transparent (alpha = 0)
- **Memory locations**:
  - P1 Palette: `0x689BA0`
  - P2 Palette: `0x689FA0`

### Palette Operations
```c
// Palette rotation (for animation effects)
void RotatePalette(int rot) {
    // Rotates entire 256-color palette by 'rot' positions
    // Used for color cycling effects
}

// Write palette to game memory
void WritePaletteToMemory(uintptr_t address, int paletteOffset) {
    // Exports 16 colors starting from paletteOffset to game memory
    // Converts back to RGB555 format
}
```

## Sprite System (.SP Files)

### File Format Specification
```c
// Sprite data is stored as 4-bit indexed color data
// Organized in 8x16 pixel tiles, packed 2 pixels per byte

struct SpriteData {
    std::vector<std::vector<uint8_t>> sprites;  // Multiple sprite pages
    int width;           // Sprite width (typically 256)
    int height;          // Sprite height (typically 256)  
    size_t currentSpriteIndex;
};
```

### Loading Algorithm
```c
// From sprite_system.cpp - LoadSpriteFile()
while (file.peek() != EOF) {
    for (int y = 0; y < 16; y++) {           // 16 rows per tile
        for (int x = 0; x < 8; x += 2) {     // 8 columns, 2 pixels per byte
            uint8_t b;
            file.read(&b, 1);
            
            // Split byte into two 4-bit pixels
            currentSprite[pos + x]     = (b & 0xF0) >> 4;  // High nibble
            currentSprite[pos + x + 1] = b & 0x0F;         // Low nibble
        }
    }
    
    // Advance to next tile (8 pixels right, or next row)
    xx += 8;
    if (xx >= importWidth) {
        xx = 0;
        yy += 16;
    }
}
```

### Sprite Memory Locations
- **Font/HUD Sprites**: `0x00456738`
- **Character Sprites**: Loaded dynamically
- **Width Options**: 64, 128, 256 pixels (configurable)

### Rendering Notes
- **4-bit indexed color**: Each pixel references palette index 0-15
- **Tile-based storage**: 8x16 pixel tiles for efficient compression
- **Multiple pages**: Large sprites split across multiple 256x256 pages

## Frame Data (.FRM Files)

### File Format Specification
```c
#pragma pack(push, 1)  // Ensure exact byte layout

struct FRMHeader {
    uint32_t NumPatterns;      // Number of animation patterns (big-endian)
    uint16_t PatternOffsets[]; // Offset to each pattern (big-endian)
};

struct PatternHeader {
    int16_t patSize;          // Number of sprite tiles in pattern (big-endian)
};

struct PatEntry {
    int16_t xOff;             // X offset (big-endian)
    int16_t yOff;             // Y offset (big-endian)
    uint8_t Pal;              // Palette and effects byte
    uint8_t padding;          // Alignment padding
    int16_t Tile;             // Sprite tile index (big-endian)
};
#pragma pack(pop)
```

### File Structure
```
FRM File Layout:
„¥„Ÿ„Ÿ Header (4 bytes)
„    „¤„Ÿ„Ÿ NumPatterns (uint32_t, big-endian)
„¥„Ÿ„Ÿ Pattern Offset Table (NumPatterns * 2 bytes)
„    „¤„Ÿ„Ÿ Offsets to each pattern (uint16_t, big-endian)
„¤„Ÿ„Ÿ Pattern Data (variable size)
    „¥„Ÿ„Ÿ Pattern 0
    „    „¥„Ÿ„Ÿ PatSize (int16_t, big-endian)
    „    „¤„Ÿ„Ÿ PatEntry[PatSize] (8 bytes each)
    „¥„Ÿ„Ÿ Pattern 1
    „    „¤„Ÿ„Ÿ ...
    „¤„Ÿ„Ÿ Pattern N
```

### Palette Byte Decoding
```c
// From sprite_system.cpp - DecodePaletteByte()
struct PaletteEffect {
    uint8_t paletteIndex;  // Lower 4 bits (0-15)
    bool mirrorH;          // Bit 6: Horizontal mirror
    bool mirrorV;          // Bit 7: Vertical mirror
};

PaletteEffect DecodePaletteByte(uint8_t palByte) {
    return {
        .paletteIndex = palByte & 0x0F,        // Bits 0-3
        .mirrorH = (palByte & 0x40) != 0,      // Bit 6
        .mirrorV = (palByte & 0x80) != 0       // Bit 7
    };
}
```

### Tile Coordinate System
```c
// Convert tile index to sprite coordinates
int spritePage = entry.Tile / 256;           // Which sprite page
int tileOffset = entry.Tile % 256;           // Tile within page
int tileX = (tileOffset % 16) * 16;          // X coordinate (16 tiles per row)
int tileY = (tileOffset / 16) * 16;          // Y coordinate (16 pixels per tile)
```

### Rendering Algorithm
```c
// Render FRM pattern (simplified)
for (int i = pattern.patSize - 1; i >= 0; i--) {  // Render back-to-front
    PatEntry& entry = pattern.entries[i];
    PaletteEffect effect = DecodePaletteByte(entry.Pal);
    
    // Calculate sprite tile position
    int tileX = (entry.Tile % 256 % 16) * 16;
    int tileY = (entry.Tile % 256 / 16) * 16;
    
    // Render 16x16 tile at position (entry.xOff, entry.yOff)
    // Apply horizontal/vertical mirroring if specified
    // Use palette index from effect.paletteIndex
}
```

## VSE System (.VSE Files)

### Complete VSE System Architecture

The VSE (Visual State Engine) is the core animation and movement system that handles **two separate data systems**:

1. **Pattern Data**: Sprite rendering, hitboxes, hurtboxes, collision data (48 bytes per frame)
2. **Movement Data**: Character velocity/movement per animation frame (2 bytes per frame)

**Critical Discovery**: These are stored in completely different locations with separate offset pointers!

### File Format Specification

```c
#pragma pack(push, 1)

// VSE Table Entry (8 bytes) - 256 entries total
struct VSETableEntry {
    int8_t pages;              // Number of animation frames in sequence
    int8_t unknown1;           // Unknown/padding
    uint8_t entryOffset_lo;    // Low byte of PATTERN data offset
    uint8_t entryOffset_hi;    // High byte of PATTERN data offset  
    int16_t unknown2;          // Unknown field
    uint8_t infoOffset_lo;     // Low byte of MOVEMENT data offset
    uint8_t infoOffset_hi;     // High byte of MOVEMENT data offset
};

// Pattern Data Structure (48 bytes) - Sprites and Combat Data
struct VSEMove {
    uint8_t sprite;            // Sprite pattern index (FRM pattern)
    int8_t duration;           // Duration in frames (1-255)
    
    // Hitbox 1 (attack box)
    int8_t hitbox1_xOff;       // X offset from player center
    int8_t hitbox1_yOff;       // Y offset from player center  
    int8_t hitbox1_Width;      // Half-width (actual width = Width * 2)
    int8_t hitbox1_Height;     // Half-height (actual height = Height * 2)
    
    // Hitbox 2 (secondary attack box)
    int8_t hitbox2_xOff;
    int8_t hitbox2_yOff;
    int8_t hitbox2_Width;
    int8_t hitbox2_Height;
    
    // Hurtbox 1 (vulnerable area)
    int8_t hurtbox1_xOff;
    int8_t hurtbox1_yOff;
    int8_t hurtbox1_Width;
    int8_t hurtbox1_Height;
    
    // Hurtbox 2 (secondary vulnerable area)
    int8_t hurtbox2_xOff;
    int8_t hurtbox2_yOff;
    int8_t hurtbox2_Width;
    int8_t hurtbox2_Height;
    
    // Collision box (character body)
    int8_t collisionbox_width;   // Half-width
    int8_t collisionbox_height;  // Half-height
    
    // Combat properties
    int8_t damageDealt;          // Damage value
    int8_t stunDealt;            // Stun/hitstun value
    int8_t blockType;            // Block type (high/low/etc.)
    int8_t hitType;              // Hit properties
    int8_t padding2;
    
    // Audio and effects
    int8_t sfx_1;                // Sound effect 1
    int8_t sfx_2;                // Sound effect 2
    int8_t padding3[2];
    
    // Additional properties
    int8_t unk_actionEntityRelated;  // Entity spawning flag
    int8_t unknownVar2;
    int8_t unknownVar3;
    
    // Clashbox (collision with opponent attacks)
    int8_t clashbox_xOff;
    int8_t clashbox_yOff;
    int8_t clashbox_Width;
    int8_t clashbox_Height;
    
    uint8_t unknownVar4;
    uint8_t padding4[11];        // Padding to 48 bytes total
};

// Movement Data Structure (2 bytes per frame)
struct VSEMovementFrame {
    int8_t x_movement;         // X velocity (-128 to +127)
    int8_t y_movement;         // Y velocity (-128 to +127)
};

// Special movement control values
#define VSE_MOVEMENT_END        0x80, 0x80  // End of animation sequence
#define VSE_MOVEMENT_JUMP(f)    0x80, (f)   // Jump to frame f

static_assert(sizeof(VSETableEntry) == 8, "VSETableEntry must be exactly 8 bytes!");
static_assert(sizeof(VSEMove) == 48, "VSEMove must be exactly 48 bytes!");
static_assert(sizeof(VSEMovementFrame) == 2, "VSEMovementFrame must be exactly 2 bytes!");
#pragma pack(pop)
```

### File Structure
```
VSE File Layout:
„¥„Ÿ„Ÿ Header/Metadata (0x0000 - 0x201F)   # 8224 bytes unknown data
„¥„Ÿ„Ÿ VSE Table (0x2020 - 0x281F)         # 256 entries ~ 8 bytes = 2048 bytes
„¥„Ÿ„Ÿ Pattern Data (0x2820+)              # Variable size, 48 bytes per move
„¤„Ÿ„Ÿ Movement Data (Variable locations)   # 2 bytes per frame, scattered throughout file
```

### VSE Table Structure (256 entries ~ 8 bytes)

**Memory Layout**: 0x2020 - 0x281F (2048 bytes total)

```c
// VSE table access
VSETableEntry* table = (VSETableEntry*)(vseFileData + 0x2020);
VSETableEntry* entry = &table[vseEntryIndex];  // vseEntryIndex: 0-255

// Key insight: Two separate offset systems!
// Bytes 2-3: Pattern data offset (sprites, hitboxes, combat data)
// Bytes 6-7: Movement data offset (velocity, position changes)
```

### Movement Data Extraction Algorithm

**Critical Discovery**: Movement data must be accessed directly from file using calculated offsets, NOT pre-parsed!

```c
// Original game algorithm (reverse engineered from 0x42fb70)
int32_t processVSEentry_movement(
    uint16_t frameIndex,        // VSE entry ID (0-255)
    const VSEData* vse_data,    // Loaded VSE file data
    uint16_t currentFrame,      // Current animation frame
    int16_t flag                // Flip flags
) {
    // 1. Access VSE table entry
    VSETableEntry* entry = &vse_data->table_entries[frameIndex];
    
    // 2. Extract movement data offset from bytes 6-7 (infoOffset)
    uint16_t table_offset = (entry->infoOffset_hi << 8) | entry->infoOffset_lo;
    uint32_t info_offset = table_offset + 0x10;  // Add base offset
    
    // 3. Calculate movement data position (2 bytes per frame)
    size_t data_index = info_offset + (currentFrame * 2);
    
    // 4. Read movement data directly from file
    int8_t x_movement = vse_data->raw_file_data[data_index];     // X velocity
    int8_t y_movement = vse_data->raw_file_data[data_index + 1]; // Y velocity
    
    // 5. Handle control flow special values
    if (x_movement == 0x80 && y_movement == 0x80) {
        // End of animation sequence
        return 0x80008080;  // Special end marker
    } else if (x_movement == 0x80) {
        // Jump to frame y_movement
        return (y_movement << 16) | 0x8080;
    }
    
    // 6. Apply horizontal flipping (two's complement)
    if (flag < 0) {  // Character facing left
        x_movement = (~x_movement) + 1;  // Assembly: "not cl; inc cl"
    }
    
    // 7. Calculate next frame
    uint16_t next_frame = currentFrame + 1;
    
    // 8. Pack result into 32-bit return value
    return (next_frame << 16) | ((x_movement & 0xFF) << 8) | (y_movement & 0xFF);
}
```

### Pattern Data Extraction

```c
// Extract 48-byte pattern data (sprites, hitboxes, combat properties)
VSEMove* GetVSEPatternData(const VSEData* vse_data, uint16_t vseEntryIndex, uint16_t frameStep) {
    VSETableEntry* entry = &vse_data->table_entries[vseEntryIndex];
    
    // Calculate pattern data offset from bytes 2-3 (entryOffset)
    uint16_t pattern_offset = (entry->entryOffset_hi << 8) | entry->entryOffset_lo;
    uint32_t absolute_offset = pattern_offset + 0x10;  // Add base offset
    
    // Each pattern move is 48 bytes
    size_t move_index = (absolute_offset - 0x2820) / 0x30 + frameStep;
    
    return &vse_data->moves[move_index];
}
```

### Complete VSE Data Structure

```c
typedef struct {
    // VSE Table (256 entries)
    VSETableEntry* table_entries;         // At file offset 0x2020
    
    // Pattern Data (48 bytes each)
    VSEMove* moves;                       // At file offset 0x2820+
    size_t num_moves;                     // Total pattern moves loaded
    uint16_t moves_base_offset;           // Pattern data base (0x2820)
    
    // Movement Data (raw file access required)
    uint8_t* raw_file_data;               // Complete VSE file in memory
    size_t total_file_size;               // Total file size
    uint16_t movement_base_offset;        // Movement calculation base (0x2010)
    
    bool is_loaded;
} VSEData;
```

### VSE Memory Locations
- **VSE Base Address**: `0x004DAAB0`
- **Table Entries**: Base + 0x0
- **Pattern Data**: Base + 0x800

### Animation Sequence Building
```c
// Complete animation sequence with both pattern and movement data
void BuildCompleteVSETimeline(VSETimelineMapping* mapping) {
    VSETableEntry* entry = &mapping->vseData->tableEntries[mapping->currentEntry];
    
    float currentTime = 0.0f;
    uint16_t currentFrame = 0;
    
    for (int step = 0; step < entry->pages; step++) {
        // Get pattern data (sprites, hitboxes)
        VSEMove* patternData = GetVSEPatternData(mapping->vseData, mapping->currentEntry, step);
        
        // Get movement data (velocity)
        int32_t movementResult = processVSEentry_movement(
            mapping->currentEntry, 
            mapping->vseData, 
            currentFrame, 
            mapping->facingDirection
        );
        
        // Extract movement values
        int8_t x_movement = (movementResult >> 8) & 0xFF;
        int8_t y_movement = movementResult & 0xFF;
        uint16_t next_frame = movementResult >> 16;
        
        // Check for control flow
        if (movementResult == 0x80008080) {
            // End of animation
            break;
        } else if ((movementResult & 0x80800000) == 0x80800000) {
            // Jump to frame
            currentFrame = next_frame;
            continue;
        }
        
        // Create complete timeline frame
        TimelineFrame frame = {
            .vseEntryIndex = mapping->currentEntry,
            .vseStepIndex = step,
            .vseMove = patternData,           // Pattern data (sprites, hitboxes)
            .x_movement = x_movement,         // Movement data (velocity)
            .y_movement = y_movement,
            .startTime = currentTime,
            .duration = patternData->duration / 60.0f,  // Convert frames to seconds
            .frameNumber = currentFrame
        };
        
        currentTime += frame.duration;
        currentFrame = next_frame;
    }
}
```

### VSE Editor Integration Notes

#### Two-System Architecture for Editor
The editor must handle both data systems independently:

```c
// Editor data structure
typedef struct {
    // Pattern editing (sprites, hitboxes, combat)
    VSEMove* currentPatternData;      // 48-byte pattern data
    int patternEditMode;              // Which pattern field is being edited
    
    // Movement editing (velocity, position)
    VSEMovementFrame* currentMovement; // 2-byte movement data
    int8_t x_velocity_preview;        // Live preview of X movement
    int8_t y_velocity_preview;        // Live preview of Y movement
    
    // Timeline integration
    float currentTimelinePosition;    // Current playback position
    bool isPlaying;                   // Animation playback state
    int currentFrame;                 // Current animation frame
    
    // Cross-system synchronization
    bool movementDataDirty;           // Movement data needs saving
    bool patternDataDirty;            // Pattern data needs saving
} VSEEditor;
```

#### Critical Editor Implementation Requirements

1. **Separate File Access**: Movement data requires direct file offset access
2. **Real-time Preview**: Changes must update both pattern and movement simultaneously  
3. **Control Flow Handling**: Editor must recognize and handle jump/end sequences
4. **Horizontal Flipping**: Preview must show correct movement for both facing directions

#### Timeline Visualization

```c
// Timeline track rendering must show both systems
typedef struct {
    uint16_t vseEntryIndex;           // VSE entry (0-255)
    uint16_t frameStep;               // Frame within animation
    
    // Pattern data (from entryOffset)
    VSEMove* patternData;             // Sprites, hitboxes, combat
    float duration;                   // Frame duration in seconds
    
    // Movement data (from infoOffset) 
    int8_t x_movement;                // X velocity this frame
    int8_t y_movement;                // Y velocity this frame
    bool isControlFlow;               // Is this a jump/end frame?
    uint16_t jumpTarget;              // Target frame for jumps
    
    // Calculated timeline position
    float startTime;                  // When this frame starts
    float endTime;                    // When this frame ends
} VSETimelineFrame;
```

#### Performance Considerations

- **Memory Usage**: Keep entire VSE file loaded for movement data access
- **Caching Strategy**: Cache frequently accessed movement sequences  
- **Real-time Updates**: Limit movement preview to 60 FPS for accuracy
- **File I/O**: Batch writes for both pattern and movement data modifications

### Duration System
- **Frame-based timing**: Duration stored as number of frames (60 FPS)
- **Range**: 1-255 frames (0.017 - 4.25 seconds)
- **Timeline conversion**: `duration_seconds = duration_frames / 60.0f`
- **Movement timing**: Movement data advances 1 frame per pattern duration cycle

## Box/Hitbox System

### Box Type Definitions
```c
enum class BoxType {
    Collision,    // Character body collision
    Hitbox1,      // Primary attack box
    Hitbox2,      // Secondary attack box  
    Hurtbox1,     // Primary vulnerable area
    Hurtbox2,     // Secondary vulnerable area
    Clashbox,     // Attack collision with opponent attacks
    Entity        // Projectile/entity boxes
};
```

### Box Data Structure
```c
struct Box {
    int16_t xOffset;         // X offset from player center
    int16_t yOffset;         // Y offset from player center
    int16_t width;           // Box width (doubled from VSE data)
    int16_t height;          // Box height (doubled from VSE data)
    BoxType type;            // Box type identifier
    ImU32 color;             // Display color
    BoxSettings* settings;   // Display settings
    bool isEntity;           // Is this an entity box?
    int16_t xPos;            // Absolute position (entities only)
    int16_t yPos;            // Absolute position (entities only)
};
```

### Coordinate System
```c
// From box_system.cpp - Camera calculation
float CalculateCameraLeftBoundary(Player* player1, Player* player2) {
    // Exact replication of game logic
    int16_t player1PosX = static_cast<int16_t>(player1->posX);
    int16_t player2PosX = static_cast<int16_t>(player2->posX);
    
    // Camera center between players, offset by 144 pixels
    int16_t cameraLeftBoundary = (player1PosX + player2PosX) / 2 - 144;
    
    // Clamp to valid range [0, 255]
    if (cameraLeftBoundary < 0) cameraLeftBoundary = 0;
    if (cameraLeftBoundary > 255) cameraLeftBoundary = 255;
    
    return static_cast<float>(cameraLeftBoundary);
}
```

### Box Coordinate Calculation
```c
// Convert box coordinates to screen coordinates
float adjustedX, adjustedY;

if (box.type == BoxType::Collision) {
    // Collision boxes are centered on player
    adjustedX = player->posX;
    adjustedY = player->posY;
} else {
    // Other boxes use offset with facing direction
    adjustedX = player->posX + (facingDirection < 0 ? -box.xOffset : box.xOffset);
    adjustedY = player->posY + box.yOffset;
}

// Convert to screen coordinates
float cameraLeftBoundary = CalculateCameraLeftBoundary(player1, player2);
float screenX = gameAreaX + (adjustedX - cameraLeftBoundary) * scaleFactor;
float screenY = gameAreaY + (adjustedY - stageFloorWorldY) * scaleFactor;
```

### Entity Box System
```c
// Entity boxes (projectiles, etc.)
for (int i = 0; i < 8; ++i) {
    uintptr_t entityAddress = entityBaseAddress + i * 0x11C;
    
    int16_t xPos = *reinterpret_cast<int16_t*>(entityAddress + 0x0);   // Absolute X
    int16_t yPos = *reinterpret_cast<int16_t*>(entityAddress + 0x2);   // Absolute Y
    int16_t width = *reinterpret_cast<int16_t*>(entityAddress + 0x26); // Width
    int16_t height = *reinterpret_cast<int16_t*>(entityAddress + 0x28);// Height
    int16_t xOffset = *reinterpret_cast<int16_t*>(entityAddress + 0x22); // EntHitXoff
    int16_t yOffset = *reinterpret_cast<int16_t*>(entityAddress + 0x24); // EntHitYoff
}
```

## Integration Notes

### Memory Addresses
```c
// Player data
#define PLAYER1_ADDRESS 0x006B2BC0
#define PLAYER2_ADDRESS 0x006B2BC0 + 0x11C

// Palette data  
#define P1_PALETTE_ADDRESS 0x689BA0
#define P2_PALETTE_ADDRESS 0x689FA0

// Entity data
#define P1_ENTITIES_ADDRESS 0x00688590
#define P2_ENTITIES_ADDRESS 0x00688e70

// Font/HUD data
#define FONT_SPRITE_ADDRESS 0x00456738
#define FONT_PALETTE_ADDRESS 0x0045653A
#define FONT_FRM_ADDRESS 0x00455E08

// VSE data
#define VSE_BASE_ADDRESS 0x004DAAB0
```

### File Path Conventions
```c
// Typical file organization
"C:/games/ml2/ncd/out/"
„¥„Ÿ„Ÿ CHAR_SP/{PREFIX}.SP       // Sprite data
„¥„Ÿ„Ÿ CHAR_PAL/{PREFIX}.PAL     // Palette data
„¥„Ÿ„Ÿ CHAR_FRM/{PREFIX}.FRM     // Frame data
„¤„Ÿ„Ÿ CHAR_VSE/{PREFIX}.VSE     // VSE animation data
```

### Endianness Notes
- **Palette files**: Little-endian (standard PC format)
- **FRM files**: Big-endian (requires byte swapping)
- **VSE files**: Little-endian for most data
- **Memory addresses**: Little-endian (x86 standard)

### Data Validation
```c
// Sanity checks implemented in existing code
if (frmData.NumPatterns > 10000) return false;     // Suspiciously large
if (patSize > 1000) return false;                  // Pattern too large
if (spriteData.sprites.size() >= 100) break;       // Too many sprites
if (width <= 0 || height <= 0) return;            // Invalid dimensions
```

## Performance Considerations

### Memory Usage
- **Sprite data**: ~64KB per 256x256 sprite page
- **Palette data**: 1KB per full palette (256 colors)
- **FRM data**: Variable, typically 1-10KB per character
- **VSE data**: Variable, typically 10-50KB per character

### Optimization Notes
- **Lazy loading**: Load sprite data only when needed
- **Texture caching**: Cache converted textures for repeated use
- **Viewport culling**: Only render visible sprites/boxes
- **LOD system**: Reduce detail based on zoom level

## Player System Structure

### Player Structure (284 bytes - 0x11C)

The Player structure contains all character state information and is used by the VSE system to control character behavior.

```c
#pragma pack(push, 1)
struct Player {
    // Basic Character Data (0x00-0x1F)
    int16_t character;             // 0x0000 Selected character ID (0-13)
    int16_t roundsWon;             // 0x0002 Number of rounds won
    int32_t hitcount;              // 0x0004 Current combo hit counter
    int16_t playerPalette;         // 0x0008 Color palette selection (0-15)
    int16_t isCPU;                 // 0x000A Flag indicating CPU control
    int16_t superMeter;            // 0x000C Special attack meter value
    int16_t superMeterStocks;      // 0x000E Special meter stock counter
    int16_t actionState;           // 0x0010 Current action state
    int16_t actionTimer;           // 0x0012 Timer for current action
    int16_t prevActionState;       // 0x0014 Previous action state
    int16_t queuedAction;          // 0x0016 Queued next action
    int16_t playerDataPtr;         // 0x0018 Pointer to player data
    int16_t health;                // 0x001A Current health (0-255)
    int16_t stunValue;             // 0x001C Current stun value
    int16_t displayHealth;         // 0x001E Health value shown in HUD
    
    // Combat State (0x20-0x3F)
    int16_t damageInvulnFrames;    // 0x0020 Damage invulnerability frames
    int16_t hitEffectCounter;      // 0x0022 Hit effect animation counter
    int16_t invulnFrames;          // 0x0024 General invulnerability frames
    int16_t padding1;              // 0x0026 Alignment padding
    int16_t defenseModifier;       // 0x0028 Defense multiplier
    int16_t facingDirection;       // 0x002A Direction (-1 = left, 1 = right)
    int16_t stateAtInteraction;    // 0x002C State during interaction
    int16_t playerFlags;           // 0x002E General player flags
    int16_t stateAtInteraction2;   // 0x0030 Secondary interaction state
    int16_t currentHitcount;       // 0x0032 Current combo hit counter
    int16_t lastHealth;            // 0x0034 Previous health value
    int16_t hitEffectCounter2;     // 0x0036 Secondary hit effect counter
    int16_t comboEffectCounter;    // 0x0038 Combo visual effect counter
    int16_t actionArray;           // 0x003A Action selection array
    int16_t padding2;              // 0x003C Padding
    int16_t currentInput;          // 0x003E Current controller input
    
    // Position & Movement (0x40-0x5F)
    int16_t posX;                  // 0x0040 Current X position (world coords)
    int16_t posY;                  // 0x0042 Current Y position (world coords)
    int16_t prevPosX;              // 0x0044 Previous X position
    int16_t prevPosY;              // 0x0046 Previous Y position
    int16_t xVelocity;             // 0x0048 X movement velocity
    int16_t yVelocity;             // 0x004A Y movement velocity
    int16_t padding3;              // 0x004C Padding
    int16_t interactionCenterY;    // 0x004E Y coordinate for interactions
    int16_t interactionCenterX;    // 0x0050 X coordinate for interactions
    int16_t currentSpriteId;       // 0x0052 Current sprite being displayed
    int16_t playerState;           // 0x0054 Overall player state (VSE entry index)
    int16_t playerPosition;        // 0x0056 Position state
    int16_t airActionTimer;        // 0x0058 Air action timer
    int16_t idleAnimTimer;         // 0x005A Idle animation timer
    int16_t stateTimer;            // 0x005C State timer
    int16_t displayedSpriteId;     // 0x005E Actually displayed sprite
    
    // Hitbox Data (0x60-0x8F) - Mirrors VSE Data
    int16_t actionStateTimer;      // 0x0060 Action state timer
    int16_t hitbox1OffsetX;        // 0x0062 Hitbox 1 X offset
    int16_t hitbox1OffsetY;        // 0x0064 Hitbox 1 Y offset
    int16_t hitbox1Width;          // 0x0066 Hitbox 1 width (doubled from VSE)
    int16_t hitbox1Height;         // 0x0068 Hitbox 1 height (doubled from VSE)
    int16_t hitbox2OffsetX;        // 0x006A Hitbox 2 X offset
    int16_t hitbox2OffsetY;        // 0x006C Hitbox 2 Y offset
    int16_t hitbox2Width;          // 0x006E Hitbox 2 width (doubled from VSE)
    int16_t hitbox2Height;         // 0x0070 Hitbox 2 height (doubled from VSE)
    int16_t hurtbox1OffsetX;       // 0x0072 Hurtbox 1 X offset
    int16_t hurtbox1OffsetY;       // 0x0074 Hurtbox 1 Y offset
    int16_t hurtbox1Width;         // 0x0076 Hurtbox 1 width (doubled from VSE)
    int16_t hurtbox1Height;        // 0x0078 Hurtbox 1 height (doubled from VSE)
    int16_t hurtbox2OffsetX;       // 0x007A Hurtbox 2 X offset
    int16_t hurtbox2OffsetY;       // 0x007C Hurtbox 2 Y offset
    int16_t hurtbox2Width;         // 0x007E Hurtbox 2 width (doubled from VSE)
    int16_t hurtbox2Height;        // 0x0080 Hurtbox 2 height (doubled from VSE)
    int16_t collisionBoxWidth;     // 0x0082 Collision box width (doubled from VSE)
    int16_t collisionBoxHeight;    // 0x0084 Collision box height (doubled from VSE)
    
    // Attack Properties (0x86-0x9F) - From VSE Data
    int16_t damageDealt;           // 0x0086 Damage dealt by attack
    int16_t stunDealt;             // 0x0088 Stun dealt by attack
    int16_t gameplayFlags;         // 0x008A Gameplay flags
    int16_t damageType;            // 0x008C Type of damage dealt
    int16_t attackProperty;        // 0x008E Attack effect flags
    int16_t sfxId;                 // 0x0090 Sound effect ID (from VSE)
    int16_t secondarySfxId;        // 0x0092 Secondary sound effect ID (from VSE)
    int16_t resetPlayerFlag;       // 0x0094 Flag for resetting player state
    int16_t jumpState;             // 0x0096 Jump state (0=ground, 1=prejump, 2=air)
    int16_t isOnGround;            // 0x0098 Ground state flag
    int16_t jumpStateRelated;      // 0x009A Jump state related flag
    int16_t collisionOffsetX;      // 0x009C Collision X offset
    int16_t collisionOffsetY;      // 0x009E Collision Y offset
    
    // Advanced State (0xA0-0xBF)
    int16_t airState;              // 0x00A0 Air state flag
    int16_t airStateRelated;       // 0x00A2 Additional air state flag
    int16_t spriteAttributes;      // 0x00A4 Sprite rendering attributes
    int16_t padding4;              // 0x00A6 Padding
    int16_t specialStateFlag;      // 0x00A8 Special state flag
    int16_t stateValue;            // 0x00AA Generic state value
    int16_t clashBoxXPos;          // 0x00AC Clash box X position
    int16_t clashBoxYPos;          // 0x00AE Clash box Y position
    int16_t clashBoxWidth;         // 0x00B0 Clash box width (doubled from VSE)
    int16_t clashBoxHeight;        // 0x00B2 Clash box height (doubled from VSE)
    int16_t specialAttackState;    // 0x00B4 Special attack state
    int16_t specialStateRelated;   // 0x00B6 Special attack related
    int16_t forceJumpTimer;        // 0x00B8 Force jump timer
    int16_t frameCounter;          // 0x00BA Frame counter
    int16_t gravityValue;          // 0x00BC Current gravity value
    int16_t usedAirAction;         // 0x00BE Air action used flag
    
    // Air Actions & Remaining Data (0xC0-0x11B)
    int16_t airActionFlag;         // 0x00C0 Air action flag
    int16_t airSpriteId;           // 0x00C2 Air sprite ID
    int16_t airActionCooldown;     // 0x00C4 Air action cooldown
    
    // Extended data (0xC6-0x11B) - Implementation specific
    uint8_t extendedData[0x56];    // Remaining bytes to 0x11C
};
static_assert(sizeof(Player) == 0x11C, "Player structure must be exactly 284 bytes");
#pragma pack(pop)
```

### Player Memory Layout

```c
// Player addresses
#define PLAYER1_ADDRESS     0x006B2BC0    // Player 1 base address
#define PLAYER2_ADDRESS     0x006B2CDC    // Player 2 base address (P1 + 0x11C)
#define PLAYER_STRUCT_SIZE  0x11C         // 284 bytes per player

// Player data access macros
#define PLAYER1() ((Player*)PLAYER1_ADDRESS)
#define PLAYER2() ((Player*)PLAYER2_ADDRESS)

// VSE-related player fields
#define PLAYER_STATE_OFFSET         0x0054  // playerState (VSE entry index)
#define PLAYER_SPRITE_ID_OFFSET     0x0052  // currentSpriteId (FRM pattern)
#define PLAYER_FACING_DIR_OFFSET    0x002A  // facingDirection
#define PLAYER_POS_X_OFFSET         0x0040  // posX
#define PLAYER_POS_Y_OFFSET         0x0042  // posY
```

### VSE-Player Integration

The VSE system directly controls many player structure fields:

```c
// VSE Move data is copied to Player structure during state transitions
void ApplyVSEMoveToPlayer(Player* player, VSEMove* move) {
    // Sprite information
    player->currentSpriteId = move->sprite;      // FRM pattern index
    player->displayedSpriteId = move->sprite;
    
    // Hitbox data (VSE stores half-width/height, player stores full)
    player->hitbox1OffsetX = move->hitbox1_xOff;
    player->hitbox1OffsetY = move->hitbox1_yOff;
    player->hitbox1Width = move->hitbox1_Width * 2;
    player->hitbox1Height = move->hitbox1_Height * 2;
    
    player->hitbox2OffsetX = move->hitbox2_xOff;
    player->hitbox2OffsetY = move->hitbox2_yOff;
    player->hitbox2Width = move->hitbox2_Width * 2;
    player->hitbox2Height = move->hitbox2_Height * 2;
    
    // Hurtbox data
    player->hurtbox1OffsetX = move->hurtbox1_xOff;
    player->hurtbox1OffsetY = move->hurtbox1_yOff;
    player->hurtbox1Width = move->hurtbox1_Width * 2;
    player->hurtbox1Height = move->hurtbox1_Height * 2;
    
    player->hurtbox2OffsetX = move->hurtbox2_xOff;
    player->hurtbox2OffsetY = move->hurtbox2_yOff;
    player->hurtbox2Width = move->hurtbox2_Width * 2;
    player->hurtbox2Height = move->hurtbox2_Height * 2;
    
    // Collision box
    player->collisionBoxWidth = move->collisionbox_width * 2;
    player->collisionBoxHeight = move->collisionbox_height * 2;
    
    // Clashbox
    player->clashBoxXPos = move->clashbox_xOff;
    player->clashBoxYPos = move->clashbox_yOff;
    player->clashBoxWidth = move->clashbox_Width * 2;
    player->clashBoxHeight = move->clashbox_Height * 2;
    
    // Combat properties
    player->damageDealt = move->damageDealt;
    player->stunDealt = move->stunDealt;
    player->damageType = move->blockType;
    player->attackProperty = move->hitType;
    
    // Audio
    player->sfxId = move->sfx_1;
    player->secondarySfxId = move->sfx_2;
}
```

## Entity System

### Entity Structure (284 bytes - 0x11C)

Each player has 8 entity slots for projectiles, effects, and other game objects:

```c
#pragma pack(push, 1)
struct Entity {
    // Position & Movement (0x00-0x1F)
    int16_t EntXpos;               // 0x0000 Current X position (absolute)
    int16_t EntYPos;               // 0x0002 Current Y position (absolute)
    int16_t tempEntXpos;           // 0x0004 Copy of X position
    int16_t tempEntYpos;           // 0x0006 Copy of Y position
    int16_t xVelocity;             // 0x0008 X movement velocity
    int16_t yVelocity;             // 0x000A Y movement velocity
    int16_t entityOwner;           // 0x000C Owner player ID (0=P1, 1=P2)
    int16_t padding1;              // 0x000E Padding
    
    // Entity Type & Animation (0x10-0x2F)
    int16_t padding2[2];           // 0x0010-0x0012 Padding
    int16_t EntityId;              // 0x0014 Entity type ID
    int16_t EntityPatFrame;        // 0x0016 Current animation frame
    int16_t entityTimer;           // 0x0018 Entity lifetime timer
    int16_t FrmPatDisp;            // 0x001A Frame pattern display
    int16_t entityUnkParam1;       // 0x001C Unknown parameter
    int16_t EntitySprite;          // 0x001E Sprite/pattern ID
    int16_t entityActionTimer;     // 0x0020 Action timer
    
    // Hitbox Data (0x22-0x4F) - Similar to Player/VSE format
    int16_t EntHitXoff;            // 0x0022 Hitbox X offset
    int16_t EntHitYoff;            // 0x0024 Hitbox Y offset
    int16_t HitboxWidth;           // 0x0026 Hitbox width
    int16_t HitboxHeight;          // 0x0028 Hitbox height
    int16_t entityHitboxXoff;      // 0x002A Secondary hitbox X offset
    int16_t entityHitbox2Yoff;     // 0x002C Secondary hitbox Y offset
    int16_t entityHitbox2Width;    // 0x002E Secondary hitbox width
    int16_t entityHitbox2Height;   // 0x0030 Secondary hitbox height
    int16_t entityHurtbox1OffsetX; // 0x0032 Hurtbox 1 X offset
    int16_t entityHurtbox1OffsetY; // 0x0034 Hurtbox 1 Y offset
    int16_t entityHurtbox1Width;   // 0x0036 Hurtbox 1 width
    int16_t entityHurtbox1Height;  // 0x0038 Hurtbox 1 height
    int16_t entityHurtbox2OffsetX; // 0x003A Hurtbox 2 X offset
    int16_t entityHurtbox2OffsetY; // 0x003C Hurtbox 2 Y offset
    int16_t entityHurtbox2Width;   // 0x003E Hurtbox 2 width
    int16_t entityHurtbox2Height;  // 0x0040 Hurtbox 2 height
    int16_t padding3;              // 0x0042 Padding
    
    // Combat Properties (0x44-0x5F)
    int16_t padding4;              // 0x0044 Padding
    int16_t EntHitDmg;             // 0x0046 Damage dealt
    int16_t EntHitStun;            // 0x0048 Stun dealt
    int16_t EntHitBlockType;       // 0x004A Block type
    int16_t EntHitType;            // 0x004C Hit type
    int16_t entityAttackProperty;  // 0x004E Attack properties
    int16_t entitySfxIDArray;      // 0x0050 Sound effect ID
    int16_t padding5;              // 0x0052 Padding
    
    // State & Behavior (0x54-0x7F)
    int16_t padding6[2];           // 0x0054-0x0056 Padding
    int16_t entityGroundState;     // 0x0058 Ground state
    int16_t entityJumpState;       // 0x005A Jump state
    int16_t entityXPosOffset;      // 0x005C X position offset
    int16_t entityYPosOffset;      // 0x005E Y position offset
    int16_t entitySpecialAttackState; // 0x0060 Special attack state
    int16_t entityStateFlag;       // 0x0062 State flags
    int16_t entitySpriteAttributes; // 0x0064 Sprite attributes
    int16_t padding7;              // 0x0066 Padding
    int16_t entityUnknownValue1;   // 0x0068 Unknown value
    int16_t entityStateValues;     // 0x006A State values
    int16_t entityFlags1;          // 0x006C Entity flags 1
    int16_t entityFlags2;          // 0x006E Entity flags 2
    int16_t entityFlags3;          // 0x0070 Entity flags 3
    int16_t entityFlags4;          // 0x0072 Entity flags 4
    int16_t padding8;              // 0x0074 Padding
    int16_t attackEffect;          // 0x0076 Attack effect
    
    // Extended Data (0x78-0x11B)
    uint8_t extendedData[0xA4];    // Remaining bytes to 0x11C
};
static_assert(sizeof(Entity) == 0x11C, "Entity structure must be exactly 284 bytes");
#pragma pack(pop)
```

### Entity Memory Layout

```c
// Entity system constants
#define NUM_ENTITIES         8           // 8 entities per player
#define ENTITY_SIZE          0x11C       // 284 bytes per entity
#define TOTAL_ENTITY_SIZE    0x8E0       // 8 * 0x11C = 2272 bytes per player

// Entity addresses
#define P1_ENTITIES_ADDRESS  0x00688590  // Player 1 entities base
#define P2_ENTITIES_ADDRESS  0x00688E70  // Player 2 entities base

// Entity access macros
#define PLAYER1_ENTITY(slot) ((Entity*)(P1_ENTITIES_ADDRESS + (slot) * ENTITY_SIZE))
#define PLAYER2_ENTITY(slot) ((Entity*)(P2_ENTITIES_ADDRESS + (slot) * ENTITY_SIZE))

// Entity slot enumeration
typedef enum {
    ENTITY_SLOT_0 = 0,    // Primary projectile
    ENTITY_SLOT_1 = 1,    // Secondary projectile
    ENTITY_SLOT_2 = 2,    // Effect 1
    ENTITY_SLOT_3 = 3,    // Effect 2
    ENTITY_SLOT_4 = 4,    // Temporary effect 1
    ENTITY_SLOT_5 = 5,    // Temporary effect 2
    ENTITY_SLOT_6 = 6,    // Special effect 1
    ENTITY_SLOT_7 = 7     // Special effect 2
} EntitySlot;
```

### Entity Slot Usage

```c
// Entity slot management
bool IsEntitySlotActive(int playerIndex, EntitySlot slot) {
    uintptr_t entityBase = (playerIndex == 0) ? P1_ENTITIES_ADDRESS : P2_ENTITIES_ADDRESS;
    Entity* entity = (Entity*)(entityBase + slot * ENTITY_SIZE);
    return entity->EntityId != 0;  // 0 = inactive
}

// Get entity for rendering
Entity* GetActiveEntity(int playerIndex, EntitySlot slot) {
    if (!IsEntitySlotActive(playerIndex, slot)) return nullptr;
    
    uintptr_t entityBase = (playerIndex == 0) ? P1_ENTITIES_ADDRESS : P2_ENTITIES_ADDRESS;
    return (Entity*)(entityBase + slot * ENTITY_SIZE);
}

// Entity types (common values)
#define ENTITY_TYPE_INACTIVE      0     // Slot is empty
#define ENTITY_TYPE_PROJECTILE    1     // Standard projectile
#define ENTITY_TYPE_FIREBALL      2     // Fireball projectile
#define ENTITY_TYPE_EFFECT        3     // Visual effect
#define ENTITY_TYPE_EXPLOSION     4     // Explosion effect
#define ENTITY_TYPE_SPECIAL       5     // Special attack effect
```

### VSE-Entity Integration

VSE moves can spawn entities through the `unk_actionEntityRelated` field:

```c
// VSE entity spawning
void ProcessVSEEntitySpawn(Player* player, VSEMove* move) {
    if (move->unk_actionEntityRelated != 0) {
        // Find available entity slot
        EntitySlot slot = FindFreeEntitySlot(player->character);
        if (slot != -1) {
            Entity* entity = GetPlayerEntity(player->character, slot);
            
            // Initialize entity from VSE data
            entity->EntityId = move->unk_actionEntityRelated;
            entity->entityOwner = player->character;
            entity->EntXpos = player->posX;
            entity->EntYPos = player->posY;
            entity->EntitySprite = move->sprite;  // Use same sprite as move
            
            // Copy combat properties
            entity->EntHitDmg = move->damageDealt;
            entity->EntHitStun = move->stunDealt;
            entity->EntHitBlockType = move->blockType;
            entity->EntHitType = move->hitType;
            entity->entitySfxIDArray = move->sfx_1;
            
            // Copy hitbox data
            entity->EntHitXoff = move->hitbox1_xOff;
            entity->EntHitYoff = move->hitbox1_yOff;
            entity->HitboxWidth = move->hitbox1_Width * 2;
            entity->HitboxHeight = move->hitbox1_Height * 2;
        }
    }
}
```

This technical documentation provides the foundation for implementing the Clay UI editor while maintaining compatibility with the existing ML2 data formats.