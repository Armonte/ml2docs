# Moon Lights 2 All-in-One Editing Tool Design Document

## Overview

This document outlines the design for a comprehensive Moon Lights 2 editing tool that integrates sprite viewing, hitbox visualization/editing, and VSE (Visual Sprite Editor) functionality into a single Clay UI-based application.

## Current State Analysis

### Existing Components
- **Sprite System**: Full sprite loading, rendering, and palette management
- **Box System**: Hitbox/hurtbox visualization and debugging
- **VSE System**: VSE file parsing and frame-by-frame editing
- **Palette System**: Color palette loading and manipulation
- **Character System**: Character data management
- **Input System**: Controller and input handling

### Current UI Framework
- ImGui-based interface with multiple windows
- SDL3 renderer integration
- Modular system architecture

## Target Architecture

### 1. Clay UI Integration

#### Core UI Structure
```
Main Application Window (Clay)
„¥„Ÿ„Ÿ Top Menu Bar
„    „¥„Ÿ„Ÿ File Menu (New, Open, Save, Export)
„    „¥„Ÿ„Ÿ View Menu (Layout options, zoom controls)
„    „¥„Ÿ„Ÿ Tools Menu (Sprite tools, hitbox tools, VSE tools)
„    „¤„Ÿ„Ÿ Help Menu
„¥„Ÿ„Ÿ Left Sidebar Panel
„    „¥„Ÿ„Ÿ Character Selector
„    „¥„Ÿ„Ÿ File Browser/Navigator
„    „¤„Ÿ„Ÿ Tool Palette
„¥„Ÿ„Ÿ Central Workspace (Tabbed Interface)
„    „¥„Ÿ„Ÿ Sprite Viewer Tab
„    „¥„Ÿ„Ÿ Hitbox Editor Tab
„    „¥„Ÿ„Ÿ VSE Editor Tab
„    „¤„Ÿ„Ÿ Animation Preview Tab
„¥„Ÿ„Ÿ Right Properties Panel
„    „¥„Ÿ„Ÿ Current Selection Properties
„    „¥„Ÿ„Ÿ Frame Information
„    „¤„Ÿ„Ÿ Export Settings
„¤„Ÿ„Ÿ Bottom Status/Timeline Bar
    „¥„Ÿ„Ÿ Current Frame Info
    „¥„Ÿ„Ÿ Timeline Scrubber
    „¤„Ÿ„Ÿ Status Messages
```

#### Clay Component Hierarchy
```c
// Main application layout
CLAY({ .id = CLAY_ID("MainWindow"), .layout = { .layoutDirection = CLAY_TOP_TO_BOTTOM } }) {
    // Top menu bar
    CLAY({ .id = CLAY_ID("MenuBar"), .layout = { .sizing = { .height = CLAY_SIZING_FIXED(32) } } }) {
        RenderMenuBar();
    }
    
    // Main content area
    CLAY({ .id = CLAY_ID("ContentArea"), .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT, .sizing = { .height = CLAY_SIZING_GROW() } } }) {
        // Left sidebar
        CLAY({ .id = CLAY_ID("LeftSidebar"), .layout = { .sizing = { .width = CLAY_SIZING_FIXED(250) } } }) {
            RenderLeftSidebar();
        }
        
        // Central workspace
        CLAY({ .id = CLAY_ID("Workspace"), .layout = { .sizing = { .width = CLAY_SIZING_GROW() } } }) {
            RenderWorkspaceTabs();
        }
        
        // Right properties panel
        CLAY({ .id = CLAY_ID("PropertiesPanel"), .layout = { .sizing = { .width = CLAY_SIZING_FIXED(300) } } }) {
            RenderPropertiesPanel();
        }
    }
    
    // Bottom status bar
    CLAY({ .id = CLAY_ID("StatusBar"), .layout = { .sizing = { .height = CLAY_SIZING_FIXED(24) } } }) {
        RenderStatusBar();
    }
}
```

### 2. Application State Management

#### Core Application State
```c
typedef struct {
    // Current workspace mode
    EditorMode currentMode; // SPRITE_VIEWER, HITBOX_EDITOR, VSE_EDITOR, ANIMATION_PREVIEW
    
    // Character and file management
    int selectedCharacterIndex;
    char currentProjectPath[260];
    bool hasUnsavedChanges;
    
    // Shared rendering state
    float globalZoom;
    Clay_Vector2 panOffset;
    bool showGrid;
    bool showDebugInfo;
    
    // Tool state
    SelectedTool currentTool;
    bool isPlaying; // For animation preview
    
    // UI state
    bool leftSidebarVisible;
    bool rightPanelVisible;
    int activeTabIndex;
    
} ML2EditorState;
```

#### Data Integration Layer
```c
typedef struct {
    // Unified data access
    SpriteSystem spriteSystem;
    BoxSystem boxSystem;
    VSESystem vseSystem;
    PaletteSystem paletteSystem;
    
    // Shared resources
    CharacterDatabase characterDB;
    ProjectSettings settings;
    
    // Clay-specific rendering data
    Clay_Arena uiArena;
    Clay_Context* clayContext;
    
} ML2EditorData;
```

### 3. Tab-Based Workspace System

#### Tab Management
```c
typedef enum {
    TAB_SPRITE_VIEWER,
    TAB_HITBOX_EDITOR,
    TAB_VSE_EDITOR,
    TAB_ANIMATION_PREVIEW
} TabType;

typedef struct {
    TabType type;
    char title[64];
    bool isDirty;
    bool isActive;
    void* tabSpecificData;
} EditorTab;

void RenderWorkspaceTabs() {
    CLAY({ .id = CLAY_ID("TabContainer"), .layout = { .layoutDirection = CLAY_TOP_TO_BOTTOM } }) {
        // Tab headers
        CLAY({ .id = CLAY_ID("TabHeaders"), .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT } }) {
            for (int i = 0; i < editorState.numTabs; i++) {
                RenderTabHeader(&editorState.tabs[i], i);
            }
        }
        
        // Active tab content
        CLAY({ .id = CLAY_ID("TabContent"), .layout = { .sizing = { .height = CLAY_SIZING_GROW() } } }) {
            switch (editorState.tabs[editorState.activeTabIndex].type) {
                case TAB_SPRITE_VIEWER:
                    RenderSpriteViewerTab();
                    break;
                case TAB_HITBOX_EDITOR:
                    RenderHitboxEditorTab();
                    break;
                case TAB_VSE_EDITOR:
                    RenderVSEEditorTab();
                    break;
                case TAB_ANIMATION_PREVIEW:
                    RenderAnimationPreviewTab();
                    break;
            }
        }
    }
}
```

### 4. Integrated Tool Panels

#### Sprite Viewer Integration
```c
void RenderSpriteViewerTab() {
    CLAY({ .id = CLAY_ID("SpriteViewer"), .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT } }) {
        // Sprite display area
        CLAY({ .id = CLAY_ID("SpriteDisplay"), .layout = { .sizing = { .width = CLAY_SIZING_GROW() } } }) {
            // Clay custom element for sprite rendering
            CLAY({ .id = CLAY_ID("SpriteCanvas"), .custom = { .customData = &spriteRenderData } }) {}
            
            // Overlay controls
            RenderSpriteOverlayControls();
        }
        
        // Sprite controls panel
        CLAY({ .id = CLAY_ID("SpriteControls"), .layout = { .sizing = { .width = CLAY_SIZING_FIXED(200) } } }) {
            RenderSpriteControls();
        }
    }
}
```

#### Hitbox Editor Integration
```c
void RenderHitboxEditorTab() {
    CLAY({ .id = CLAY_ID("HitboxEditor"), .layout = { .layoutDirection = CLAY_TOP_TO_BOTTOM } }) {
        // Toolbar
        CLAY({ .id = CLAY_ID("HitboxToolbar"), .layout = { .sizing = { .height = CLAY_SIZING_FIXED(40) } } }) {
            RenderHitboxToolbar();
        }
        
        // Main editing area
        CLAY({ .id = CLAY_ID("HitboxWorkspace"), .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT, .sizing = { .height = CLAY_SIZING_GROW() } } }) {
            // Hitbox visualization
            CLAY({ .id = CLAY_ID("HitboxCanvas"), .custom = { .customData = &hitboxRenderData } }) {}
            
            // Hitbox properties
            CLAY({ .id = CLAY_ID("HitboxProperties"), .layout = { .sizing = { .width = CLAY_SIZING_FIXED(250) } } }) {
                RenderHitboxProperties();
            }
        }
    }
}
```

#### VSE Editor Integration
```c
void RenderVSEEditorTab() {
    CLAY({ .id = CLAY_ID("VSEEditor"), .layout = { .layoutDirection = CLAY_TOP_TO_BOTTOM } }) {
        // VSE timeline
        CLAY({ .id = CLAY_ID("VSETimeline"), .layout = { .sizing = { .height = CLAY_SIZING_FIXED(100) } } }) {
            RenderVSETimeline();
        }
        
        // VSE content area
        CLAY({ .id = CLAY_ID("VSEContent"), .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT, .sizing = { .height = CLAY_SIZING_GROW() } } }) {
            // Frame preview
            CLAY({ .id = CLAY_ID("VSEPreview"), .custom = { .customData = &vseRenderData } }) {}
            
            // Frame data editor
            CLAY({ .id = CLAY_ID("VSEFrameData"), .layout = { .sizing = { .width = CLAY_SIZING_FIXED(300) } } }) {
                RenderVSEFrameDataEditor();
            }
        }
    }
}
```

### 5. Unified Rendering System

#### Custom Clay Renderer Integration
```c
typedef struct {
    SDL_Renderer* sdlRenderer;
    SDL_Texture* spriteTexture;
    BoxSystem* boxSystem;
    SpriteSystem* spriteSystem;
    VSESystem* vseSystem;
} ML2ClayRenderer;

void RenderClayCustomElements(Clay_RenderCommand* command) {
    switch (command->commandType) {
        case CLAY_RENDER_COMMAND_TYPE_CUSTOM: {
            CustomRenderData* data = (CustomRenderData*)command->renderData.custom.customData;
            
            switch (data->type) {
                case CUSTOM_ELEMENT_SPRITE:
                    RenderSpriteElement(data, command->boundingBox);
                    break;
                case CUSTOM_ELEMENT_HITBOX:
                    RenderHitboxElement(data, command->boundingBox);
                    break;
                case CUSTOM_ELEMENT_VSE_FRAME:
                    RenderVSEFrameElement(data, command->boundingBox);
                    break;
            }
            break;
        }
        // Handle other Clay render commands...
    }
}
```

### 6. Tool Integration Features

#### Unified Zoom and Pan
```c
typedef struct {
    float zoomLevel;
    Clay_Vector2 panOffset;
    Clay_BoundingBox viewportBounds;
} ViewportState;

void HandleViewportInput(ViewportState* viewport) {
    // Mouse wheel zoom
    if (mouseWheelDelta != 0) {
        viewport->zoomLevel *= (1.0f + mouseWheelDelta * 0.1f);
        viewport->zoomLevel = ClampFloat(viewport->zoomLevel, 0.1f, 10.0f);
    }
    
    // Pan with middle mouse button
    if (isMiddleMouseDown) {
        viewport->panOffset.x += mouseDelta.x;
        viewport->panOffset.y += mouseDelta.y;
    }
}
```

#### Cross-Tool Data Sharing
```c
typedef struct {
    int currentFrame;
    int currentPattern;
    int selectedCharacter;
    PaletteData* activePalette;
    
    // Shared between tools
    bool showCollisionBoxes;
    bool showHitboxes;
    bool showHurtboxes;
    bool showSprites;
    
} SharedToolState;
```

### 7. Project Management

#### Project File Format
```c
typedef struct {
    char projectName[64];
    char characterName[32];
    char basePath[260];
    
    // Tool-specific settings
    SpriteViewerSettings spriteSettings;
    HitboxEditorSettings hitboxSettings;
    VSEEditorSettings vseSettings;
    
    // Export settings
    ExportSettings exportSettings;
    
} ProjectFile;
```

#### File Operations
```c
bool SaveProject(const char* filepath, ProjectFile* project);
bool LoadProject(const char* filepath, ProjectFile* project);
bool ExportCharacterData(const char* outputPath, ExportSettings* settings);
```

### 8. Implementation Phases

#### Phase 1: Clay UI Foundation
1. Set up Clay UI integration with SDL3
2. Create basic layout structure
3. Implement tab system
4. Basic menu system

#### Phase 2: Tool Integration
1. Port sprite viewer to Clay
2. Port hitbox editor to Clay
3. Port VSE editor to Clay
4. Implement custom rendering elements

#### Phase 3: Unified Features
1. Cross-tool data sharing
2. Unified viewport controls
3. Project management system
4. Export functionality

#### Phase 4: Polish and Features
1. Animation preview system
2. Advanced editing tools
3. Batch operations
4. Plugin system

### 9. Technical Considerations

#### Memory Management
- Single Clay arena for all UI elements
- Separate arenas for tool-specific data
- Efficient texture management for sprites

#### Performance
- Viewport culling for large sprites
- Lazy loading of character data
- Efficient redraw only when needed

#### File I/O
- Async file loading for better UX
- Auto-save functionality
- Version control integration

### 10. User Experience Goals

#### Workflow Integration
- Seamless switching between tools
- Consistent keyboard shortcuts
- Drag and drop support

#### Visual Consistency
- Unified color scheme
- Consistent iconography
- Professional appearance

#### Accessibility
- Keyboard navigation
- Scalable UI elements
- Color blind friendly palettes

## Conclusion

This design creates a comprehensive, professional editing tool that leverages the existing codebase while providing a modern, unified interface using Clay UI. The modular architecture allows for incremental development while maintaining the functionality of each individual tool.