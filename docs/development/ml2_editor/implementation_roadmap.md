# Moon Lights 2 Editor Implementation Roadmap

## Overview

This document provides a comprehensive implementation roadmap for building the Moon Lights 2 all-in-one editing tool. The project transforms individual ImGui-based tools into a unified Clay UI-based professional editing environment with timeline, state machine, and animation capabilities.

## Project Structure

```
ML2Editor/
„¥„Ÿ„Ÿ src/
„    „¥„Ÿ„Ÿ ml2_editor_main.c           # Main application entry point
„    „¥„Ÿ„Ÿ ml2_editor.h                # Core application header
„    „¥„Ÿ„Ÿ timeline/                   # Timeline system
„    „    „¥„Ÿ„Ÿ timeline_system.h/c     # Core timeline functionality
„    „    „¥„Ÿ„Ÿ animation_player.h/c    # Animation playback engine
„    „    „¤„Ÿ„Ÿ state_machine.h/c       # State machine editor
„    „¥„Ÿ„Ÿ ui/                         # UI components
„    „    „¥„Ÿ„Ÿ ui_common.h             # Shared UI definitions
„    „    „¥„Ÿ„Ÿ menu_bar.c              # Main menu bar
„    „    „¥„Ÿ„Ÿ sidebar.c               # Left sidebar (character list)
„    „    „¥„Ÿ„Ÿ properties_panel.c      # Right properties panel
„    „    „¥„Ÿ„Ÿ timeline_ui.c           # Timeline interface
„    „    „¤„Ÿ„Ÿ state_machine_ui.c      # State machine interface
„    „¤„Ÿ„Ÿ utils/                      # Utility functions
„        „¥„Ÿ„Ÿ utils.h                 # Common utilities
„        „¥„Ÿ„Ÿ file_utils.c            # File I/O operations
„        „¥„Ÿ„Ÿ math_utils.c            # Math and interpolation
„        „¤„Ÿ„Ÿ memory_utils.c          # Memory management
„¥„Ÿ„Ÿ mlfixtest/                      # Existing systems (to be integrated)
„    „¥„Ÿ„Ÿ menu/impl/                  # Current ImGui-based tools
„    „    „¥„Ÿ„Ÿ sprite_system.cpp/h     # Sprite viewer
„    „    „¥„Ÿ„Ÿ box_system.cpp/h        # Hitbox editor  
„    „    „¥„Ÿ„Ÿ vse.cpp/h               # VSE editor
„    „    „¥„Ÿ„Ÿ palette_system.cpp/h    # Palette management
„    „    „¤„Ÿ„Ÿ character_info.cpp/h    # Character data
„¥„Ÿ„Ÿ clay/                           # Clay UI library
„    „¥„Ÿ„Ÿ clay.h                      # Main Clay header
„    „¤„Ÿ„Ÿ renderers/SDL3/             # SDL3 renderer
„¥„Ÿ„Ÿ assets/                         # Asset files
„    „¥„Ÿ„Ÿ fonts/                      # UI fonts
„    „¥„Ÿ„Ÿ icons/                      # UI icons
„    „¤„Ÿ„Ÿ themes/                     # Color themes
„¥„Ÿ„Ÿ docs/                           # Documentation
„    „¥„Ÿ„Ÿ ML2_EDITING_TOOL_DESIGN.md      # Main design document
„    „¥„Ÿ„Ÿ ML2_TIMELINE_SYSTEM_DESIGN.md   # Timeline system design
„    „¤„Ÿ„Ÿ ML2_IMPLEMENTATION_ROADMAP.md   # This document
„¥„Ÿ„Ÿ CMakeLists.txt                  # Build configuration
„¤„Ÿ„Ÿ README.md                       # Project overview
```

## Implementation Phases

### Phase 1: Foundation Setup (Week 1-2)

#### 1.1 Project Setup
- [ ] Set up CMake build system
- [ ] Integrate Clay UI library
- [ ] Configure SDL3 dependencies
- [ ] Create basic project structure
- [ ] Set up Git repository and workflow

#### 1.2 Basic Clay UI Integration
```c
// Milestone: Basic Clay window with SDL3 renderer
int main() {
    InitializeSDL();
    InitializeClay();
    
    while(running) {
        HandleEvents();
        RenderBasicClayInterface();
    }
    
    return 0;
}
```

#### 1.3 Core Application Framework
- [ ] Create main application state structure
- [ ] Implement basic event handling
- [ ] Set up memory management with Clay arenas
- [ ] Create error handling system

**Deliverable**: Basic Clay UI application that displays a window with placeholder interface.

### Phase 2: Core UI Layout (Week 3-4)

#### 2.1 Main Layout Implementation
```c
// Milestone: Professional IDE-like layout
void RenderMainLayout() {
    CLAY({ .layout = { .layoutDirection = CLAY_TOP_TO_BOTTOM } }) {
        RenderMenuBar();        // File, Edit, View, Tools, Help
        RenderMainContent();    // Left sidebar + workspace + properties
        RenderStatusBar();      // Status information
    }
}
```

#### 2.2 Tab System
- [ ] Implement tab management system
- [ ] Create tab switching functionality
- [ ] Add tab-specific state management
- [ ] Implement tab close/open functionality

#### 2.3 Panel System
- [ ] Left sidebar (character selection)
- [ ] Right properties panel
- [ ] Resizable panels with drag handles
- [ ] Panel visibility toggles

**Deliverable**: Complete IDE-like interface with working tabs and panels.

### Phase 3: Tool Integration (Week 5-8)

#### 3.1 Sprite Viewer Integration
```c
// Milestone: Working sprite viewer in Clay UI
void RenderSpriteViewerTab() {
    CLAY({ .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT } }) {
        // Sprite display area
        CLAY({ .custom = { .customData = &spriteRenderData } }) {
            // Integrate existing SpriteSystem rendering
        }
        
        // Controls panel
        RenderSpriteControls();
    }
}
```

Tasks:
- [ ] Port SpriteSystem to Clay custom elements
- [ ] Implement sprite loading and display
- [ ] Add zoom and pan controls
- [ ] Integrate palette system
- [ ] Add sprite export functionality

#### 3.2 Hitbox Editor Integration
```c
// Milestone: Visual hitbox editing with real-time updates
void RenderHitboxEditorTab() {
    CLAY({ .layout = { .layoutDirection = CLAY_TOP_TO_BOTTOM } }) {
        RenderHitboxToolbar();
        
        CLAY({ .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT } }) {
            // Hitbox visualization
            CLAY({ .custom = { .customData = &hitboxRenderData } }) {}
            
            // Properties panel
            RenderHitboxProperties();
        }
    }
}
```

Tasks:
- [ ] Port BoxSystem to Clay custom elements
- [ ] Implement box selection and editing
- [ ] Add visual box manipulation (drag, resize)
- [ ] Integrate with sprite display
- [ ] Add box property editing

#### 3.3 VSE Editor Integration
- [ ] Port VSE system to Clay UI
- [ ] Implement frame navigation
- [ ] Add VSE data editing interface
- [ ] Create frame property editor
- [ ] Add VSE export functionality

**Deliverable**: All three main tools working within the unified interface.

### Phase 4: Timeline System (Week 9-12)

#### 4.1 Core Timeline Architecture
```c
// Milestone: Basic timeline with frame display
typedef struct {
    float currentTime;
    bool isPlaying;
    AnimationSequence* currentSequence;
    TimelineTrack tracks[MAX_TIMELINE_TRACKS];
} TimelineState;

void RenderTimelineTab() {
    CLAY({ .layout = { .layoutDirection = CLAY_TOP_TO_BOTTOM } }) {
        RenderTimelineToolbar();    // Play, pause, stop controls
        RenderTimelineMain();       // Track list + timeline canvas
        RenderTimelineControls();   // Scrubber and frame info
    }
}
```

Tasks:
- [ ] Implement timeline data structures
- [ ] Create timeline rendering system
- [ ] Add playback controls (play, pause, stop)
- [ ] Implement timeline scrubbing
- [ ] Add frame snapping

#### 4.2 Multi-Track System
```c
// Milestone: Multiple tracks for different data types
typedef enum {
    TRACK_TYPE_SPRITE,
    TRACK_TYPE_HITBOX,
    TRACK_TYPE_HURTBOX,
    TRACK_TYPE_AUDIO,
    TRACK_TYPE_EFFECT
} TimelineTrackType;
```

Tasks:
- [ ] Implement track management
- [ ] Create track-specific rendering
- [ ] Add track visibility controls
- [ ] Implement track grouping
- [ ] Add multi-track selection

#### 4.3 VSE Integration
```c
// Milestone: VSE data displayed as timeline
void BuildTimelineFromVSE(VSEData* vseData, TimelineState* timeline) {
    // Convert VSE pages to timeline frames
    // Map VSE duration bytes to timeline
    // Create keyframes for hitboxes, sprites, etc.
}
```

Tasks:
- [ ] Map VSE data to timeline format
- [ ] Implement VSE duration handling
- [ ] Create automatic keyframe generation
- [ ] Add VSE-specific timeline controls

**Deliverable**: Working timeline system with VSE integration and multi-track support.

### Phase 5: Animation Playback (Week 13-16)

#### 5.1 Animation Player Engine
```c
// Milestone: Frame-accurate animation playback
typedef struct {
    AnimationSequence* currentSequence;
    float frameTimer;
    bool isPlaying;
    float speed;
    
    // Callbacks for frame events
    void (*onFrameChanged)(int frame, void* userData);
    void (*onHitboxActive)(HitboxData* hitbox, void* userData);
} AnimationPlayer;
```

Tasks:
- [ ] Implement animation player core
- [ ] Add frame-accurate timing
- [ ] Create playback speed controls
- [ ] Implement loop handling
- [ ] Add frame event callbacks

#### 5.2 State Visualization
- [ ] Render current frame sprite
- [ ] Display active hitboxes/hurtboxes
- [ ] Show frame duration indicators
- [ ] Add onion skinning (previous/next frames)
- [ ] Implement smooth interpolation between frames

#### 5.3 Timeline Interaction
- [ ] Timeline scrubbing updates animation
- [ ] Animation playback moves timeline cursor
- [ ] Frame selection in timeline
- [ ] Keyframe editing updates animation

**Deliverable**: Complete animation playback system with timeline integration.

### Phase 6: State Machine Editor (Week 17-20)

#### 6.1 State Graph Visualization
```c
// Milestone: Visual state machine graph
typedef struct {
    char name[32];
    AnimationSequence* animation;
    Clay_Vector2 graphPosition;
    StateTransition* transitions;
    int numTransitions;
} CharacterState;

void RenderStateMachineTab() {
    CLAY({ .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT } }) {
        RenderStateList();      // List of states
        RenderStateGraph();     // Visual state graph
        RenderStateProperties(); // Properties panel
    }
}
```

Tasks:
- [ ] Implement state data structures
- [ ] Create visual state graph rendering
- [ ] Add state positioning and layout
- [ ] Implement state selection
- [ ] Add state creation/deletion

#### 6.2 Transition System
```c
// Milestone: Visual transition editing
typedef struct {
    CharacterState* fromState;
    CharacterState* toState;
    TransitionCondition condition;
    int transitionFrames;
} StateTransition;
```

Tasks:
- [ ] Implement transition visualization
- [ ] Add transition creation (drag between states)
- [ ] Create transition condition editor
- [ ] Add transition timing controls
- [ ] Implement transition validation

#### 6.3 Integration with Timeline
- [ ] Timeline shows state-specific animations
- [ ] State selection updates timeline
- [ ] Timeline editing updates state animations
- [ ] State transition previews

**Deliverable**: Complete state machine editor with visual graph and timeline integration.

### Phase 7: Advanced Features (Week 21-24)

#### 7.1 Project Management
```c
// Milestone: Save/load complete projects
typedef struct {
    char projectName[64];
    char characterName[32];
    
    // All tool settings
    SpriteViewerSettings spriteSettings;
    HitboxEditorSettings hitboxSettings;
    VSEEditorSettings vseSettings;
    TimelineSettings timelineSettings;
} ProjectFile;
```

Tasks:
- [ ] Implement project file format
- [ ] Add save/load functionality
- [ ] Create recent projects list
- [ ] Add auto-save feature
- [ ] Implement project templates

#### 7.2 Export System
- [ ] Export VSE files (modified)
- [ ] Export sprite sheets
- [ ] Export hitbox data
- [ ] Export state machine definitions
- [ ] Export animation sequences

#### 7.3 Advanced Timeline Features
- [ ] Keyframe interpolation curves
- [ ] Timeline markers and regions
- [ ] Multi-selection editing
- [ ] Copy/paste keyframes
- [ ] Timeline zoom and navigation

**Deliverable**: Complete professional editing tool with project management and export capabilities.

### Phase 8: Polish and Optimization (Week 25-28)

#### 8.1 Performance Optimization
- [ ] Timeline rendering optimization
- [ ] Memory usage optimization
- [ ] Large project handling
- [ ] Lazy loading of assets
- [ ] Background processing

#### 8.2 User Experience
- [ ] Keyboard shortcuts
- [ ] Context menus
- [ ] Drag and drop support
- [ ] Undo/redo system
- [ ] Customizable interface

#### 8.3 Documentation and Testing
- [ ] User manual
- [ ] Developer documentation
- [ ] Unit tests for core systems
- [ ] Integration tests
- [ ] Performance benchmarks

**Deliverable**: Production-ready editor with professional polish and documentation.

## Technical Milestones

### Key Integration Points

1. **Clay Custom Elements**: Bridge between Clay UI and existing rendering systems
2. **Shared State Management**: Ensure all tools work with the same data
3. **VSE Timeline Mapping**: Convert VSE duration system to timeline
4. **Animation Player**: Frame-accurate playback with callbacks
5. **State Machine Graph**: Visual editing of character states

### Performance Targets

- **Timeline Rendering**: 60 FPS with 1000+ keyframes
- **Animation Playback**: Frame-perfect timing at 60 FPS
- **Memory Usage**: < 100MB for typical projects
- **Load Times**: < 2 seconds for character data
- **UI Responsiveness**: < 16ms frame time

### Testing Strategy

- **Unit Tests**: Core timeline and animation systems
- **Integration Tests**: Tool interaction and data sharing
- **Performance Tests**: Large project handling
- **User Testing**: Professional animators feedback
- **Compatibility Tests**: Various character data formats

## Risk Mitigation

### High Risk Items
1. **Clay UI Learning Curve**: Mitigate with early prototyping
2. **Performance with Large Timelines**: Implement LOD system early
3. **VSE Data Complexity**: Maintain backward compatibility
4. **State Machine Complexity**: Start with simple graph editor

### Fallback Plans
- Keep existing ImGui tools as reference
- Modular architecture allows independent development
- Timeline system can work independently of state machine
- Export functionality ensures data portability

## Success Criteria

### MVP (Minimum Viable Product)
- [ ] All three tools integrated into unified interface
- [ ] Basic timeline with VSE integration
- [ ] Animation playback with frame accuracy
- [ ] Project save/load functionality

### Professional Release
- [ ] Complete state machine editor
- [ ] Advanced timeline features
- [ ] Export to all necessary formats
- [ ] Professional UI polish
- [ ] Comprehensive documentation

### Future Enhancements
- [ ] Plugin system for custom tools
- [ ] Collaborative editing features
- [ ] Version control integration
- [ ] Scripting system for automation
- [ ] 3D preview integration

## Conclusion

This roadmap provides a clear path from the current ImGui-based tools to a professional Clay UI-based editor with timeline and state machine capabilities. The phased approach allows for incremental development while maintaining working tools throughout the process.

The key to success will be maintaining the modular architecture, ensuring proper integration points between systems, and focusing on professional-quality user experience throughout development.