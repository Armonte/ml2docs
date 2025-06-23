# Moon Lights 2 Clay UI Integration Design

## Overview

This document details how to integrate Clay UI with the existing ML2 file format systems to create a professional all-in-one editor. The design leverages Clay's custom element system to render game-specific data while maintaining compatibility with existing code.

## Clay UI Architecture Integration

### Custom Element System

Clay UI allows custom rendering elements that can display complex game data. We'll create specialized renderers for ML2 data types:

```c
// Clay custom element types for ML2
typedef enum {
    ML2_ELEMENT_SPRITE_VIEWER,      // Renders sprite with palette
    ML2_ELEMENT_HITBOX_OVERLAY,     // Renders hitboxes over sprite
    ML2_ELEMENT_TIMELINE_TRACK,     // Timeline with VSE frames
    ML2_ELEMENT_PALETTE_STRIP,      // 16-color palette display
    ML2_ELEMENT_FRM_PATTERN,        // Individual FRM pattern preview
    ML2_ELEMENT_VSE_FRAME_DATA,     // VSE frame property editor
    ML2_ELEMENT_CHARACTER_PORTRAIT  // Character selection thumbnail
} ML2_CustomElementType;

// Custom element data payload
typedef struct {
    ML2_CustomElementType type;
    union {
        struct {
            SpriteData* spriteData;
            PaletteData* paletteData;
            int currentSpriteIndex;
            int currentPaletteIndex;
            float zoom;
            Clay_Vector2 panOffset;
        } spriteViewer;
        
        struct {
            VSEData* vseData;
            int currentEntry;
            int currentStep;
            float currentTime;
            bool isPlaying;
            Box* boxes;
            int boxCount;
        } hitboxOverlay;
        
        struct {
            VSETimelineMapping* timeline;
            float playheadPosition;
            float zoomLevel;
            float scrollOffset;
            bool showHitboxTrack;
            bool showAudioTrack;
        } timelineTrack;
        
        struct {
            ImVec4* colors;
            int paletteIndex;
            int selectedColor;
        } paletteStrip;
    };
} ML2_CustomElementData;
```

### Layout System

Clay's powerful layout system allows us to create a professional editor layout:

```c
// Main editor layout structure
void RenderML2Editor() {
    // Main container with dark theme
    CLAY(CLAY_ID("MainEditor"), 
         CLAY_LAYOUT(.sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_GROW()}),
         CLAY_RECTANGLE(.color = {0.1f, 0.1f, 0.1f, 1.0f})) {
        
        // Top menu bar
        RenderMenuBar();
        
        // Main content area (horizontal split)
        CLAY(CLAY_ID("ContentArea"), 
             CLAY_LAYOUT(.direction = CLAY_LEFT_TO_RIGHT, .sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_GROW()})) {
            
            // Left sidebar - Character/File browser
            RenderSidebar();
            
            // Center area - Tabbed workspace
            RenderWorkspace();
            
            // Right sidebar - Properties panel
            RenderPropertiesPanel();
        }
        
        // Bottom panel - Timeline
        RenderTimelinePanel();
    }
}
```

### Tab System Implementation

```c
typedef enum {
    TAB_SPRITE_VIEWER,
    TAB_HITBOX_EDITOR, 
    TAB_VSE_EDITOR,
    TAB_ANIMATION_PREVIEW,
    TAB_PALETTE_EDITOR,
    TAB_STATE_MACHINE
} TabType;

typedef struct {
    TabType activeTab;
    CharacterInfo* currentCharacter;
    SpriteData spriteData;
    PaletteData paletteData;
    FRMData frmData;
    VSEData vseData;
    VSETimelineMapping timeline;
    float playbackTime;
    bool isPlaying;
} EditorState;

void RenderWorkspace() {
    CLAY(CLAY_ID("Workspace"), 
         CLAY_LAYOUT(.sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_GROW()})) {
        
        // Tab bar
        CLAY(CLAY_ID("TabBar"), 
             CLAY_LAYOUT(.direction = CLAY_LEFT_TO_RIGHT, .sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_FIXED(30)})) {
            
            RenderTab("Sprite Viewer", TAB_SPRITE_VIEWER);
            RenderTab("Hitbox Editor", TAB_HITBOX_EDITOR); 
            RenderTab("VSE Editor", TAB_VSE_EDITOR);
            RenderTab("Animation Preview", TAB_ANIMATION_PREVIEW);
            RenderTab("Palette Editor", TAB_PALETTE_EDITOR);
            RenderTab("State Machine", TAB_STATE_MACHINE);
        }
        
        // Tab content
        CLAY(CLAY_ID("TabContent"), 
             CLAY_LAYOUT(.sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_GROW()})) {
            
            switch(editorState.activeTab) {
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
                case TAB_PALETTE_EDITOR:
                    RenderPaletteEditorTab();
                    break;
                case TAB_STATE_MACHINE:
                    RenderStateMachineTab();
                    break;
            }
        }
    }
}
```

## Custom Element Implementations

### Sprite Viewer Custom Element

```c
// Custom sprite renderer that integrates with existing sprite system
void Clay_RenderCustomElement_SpriteViewer(Clay_RenderCommandArray* renderCommands, 
                                          Clay_RenderCommand* renderCommand, 
                                          ML2_CustomElementData* elementData) {
    
    auto* data = &elementData->spriteViewer;
    
    // Get current sprite and palette
    if (!data->spriteData || !data->paletteData) return;
    
    std::vector<uint8_t>& currentSprite = data->spriteData->sprites[data->currentSpriteIndex];
    std::vector<ImVec4>& currentPalette = data->paletteData->palettes[data->currentPaletteIndex];
    
    // Calculate render area
    Clay_BoundingBox bounds = renderCommand->boundingBox;
    float centerX = bounds.x + bounds.width / 2.0f;
    float centerY = bounds.y + bounds.height / 2.0f;
    
    // Apply zoom and pan
    float spriteWidth = data->spriteData->width * data->zoom;
    float spriteHeight = data->spriteData->height * data->zoom;
    float renderX = centerX - spriteWidth / 2.0f + data->panOffset.x;
    float renderY = centerY - spriteHeight / 2.0f + data->panOffset.y;
    
    // Create texture from indexed sprite data
    uint32_t* pixelBuffer = CreateTextureFromSprite(currentSprite, currentPalette, 
                                                   data->spriteData->width, data->spriteData->height);
    
    // Render to screen using Clay's renderer
    Clay_Renderer_RenderTexture(renderCommands, pixelBuffer, 
                               renderX, renderY, spriteWidth, spriteHeight);
    
    free(pixelBuffer);
}
```

### Hitbox Overlay Custom Element

```c
void Clay_RenderCustomElement_HitboxOverlay(Clay_RenderCommandArray* renderCommands,
                                           Clay_RenderCommand* renderCommand,
                                           ML2_CustomElementData* elementData) {
    
    auto* data = &elementData->hitboxOverlay;
    
    if (!data->vseData || !data->boxes) return;
    
    Clay_BoundingBox bounds = renderCommand->boundingBox;
    
    // Get current VSE frame
    VSEMove* currentMove = GetCurrentVSEMove(data->vseData, data->currentEntry, data->currentStep);
    if (!currentMove) return;
    
    // Render hitboxes with proper scaling and positioning
    for (int i = 0; i < data->boxCount; i++) {
        Box* box = &data->boxes[i];
        
        // Calculate screen coordinates (reuse existing box_system logic)
        float screenX, screenY, screenWidth, screenHeight;
        CalculateBoxScreenCoordinates(box, bounds, &screenX, &screenY, &screenWidth, &screenHeight);
        
        // Render box with appropriate color
        Clay_Color boxColor = GetBoxTypeColor(box->type);
        Clay_Renderer_RenderRectangleOutline(renderCommands, screenX, screenY, 
                                           screenWidth, screenHeight, 2.0f, boxColor);
        
        // Render box label
        if (box->type != BoxType::Collision) {
            char label[32];
            snprintf(label, sizeof(label), "%s", GetBoxTypeName(box->type));
            Clay_Renderer_RenderText(renderCommands, label, screenX, screenY - 15, boxColor);
        }
    }
}
```

### Timeline Track Custom Element

```c
void Clay_RenderCustomElement_TimelineTrack(Clay_RenderCommandArray* renderCommands,
                                           Clay_RenderCommand* renderCommand, 
                                           ML2_CustomElementData* elementData) {
    
    auto* data = &elementData->timelineTrack;
    
    if (!data->timeline) return;
    
    Clay_BoundingBox bounds = renderCommand->boundingBox;
    
    // Timeline background
    Clay_Renderer_RenderRectangle(renderCommands, bounds.x, bounds.y, bounds.width, bounds.height,
                                (Clay_Color){0.15f, 0.15f, 0.15f, 1.0f});
    
    // Draw timeline frames
    VSETimelineMapping* timeline = data->timeline;
    float pixelsPerSecond = 100.0f * data->zoomLevel;
    float startTime = data->scrollOffset;
    float endTime = startTime + (bounds.width / pixelsPerSecond);
    
    // Draw frame blocks
    for (auto& frame : timeline->timelineFrames) {
        if (frame.startTime > endTime || (frame.startTime + frame.duration) < startTime) continue;
        
        float frameX = bounds.x + (frame.startTime - startTime) * pixelsPerSecond;
        float frameWidth = frame.duration * pixelsPerSecond;
        
        // Frame background
        Clay_Color frameColor = {0.3f, 0.5f, 0.8f, 0.8f};
        Clay_Renderer_RenderRectangle(renderCommands, frameX, bounds.y + 5, 
                                    frameWidth, bounds.height - 10, frameColor);
        
        // Frame border
        Clay_Renderer_RenderRectangleOutline(renderCommands, frameX, bounds.y + 5,
                                           frameWidth, bounds.height - 10, 1.0f,
                                           (Clay_Color){0.5f, 0.7f, 1.0f, 1.0f});
        
        // Frame label (VSE entry and step)
        char label[32];
        snprintf(label, sizeof(label), "%d.%d", frame.vseEntryIndex, frame.vseStepIndex);
        Clay_Renderer_RenderText(renderCommands, label, frameX + 2, bounds.y + 8,
                               (Clay_Color){1.0f, 1.0f, 1.0f, 1.0f});
    }
    
    // Draw playhead
    float playheadX = bounds.x + (data->playheadPosition - startTime) * pixelsPerSecond;
    if (playheadX >= bounds.x && playheadX <= bounds.x + bounds.width) {
        Clay_Renderer_RenderLine(renderCommands, playheadX, bounds.y, playheadX, bounds.y + bounds.height,
                               2.0f, (Clay_Color){1.0f, 0.2f, 0.2f, 1.0f});
    }
    
    // Draw hitbox track if enabled
    if (data->showHitboxTrack) {
        RenderHitboxTrack(renderCommands, bounds, timeline, data);
    }
}
```

## Specific Tab Implementations

### Sprite Viewer Tab

```c
void RenderSpriteViewerTab() {
    CLAY(CLAY_ID("SpriteViewerTab"), 
         CLAY_LAYOUT(.sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_GROW()})) {
        
        // Main sprite display (custom element)
        CLAY(CLAY_ID("SpriteDisplay"), 
             CLAY_LAYOUT(.sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_GROW()}),
             CLAY_CUSTOM_ELEMENT(.customElementType = ML2_ELEMENT_SPRITE_VIEWER,
                                .customElementData = &spriteViewerData)) {}
        
        // Overlay controls
        CLAY(CLAY_ID("SpriteControls"), 
             CLAY_LAYOUT(.direction = CLAY_LEFT_TO_RIGHT, .sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_FIXED(40)})) {
            
            // Sprite navigation
            if (RenderButton("Previous Sprite")) {
                if (editorState.spriteData.currentSpriteIndex > 0) {
                    editorState.spriteData.currentSpriteIndex--;
                }
            }
            
            // Current sprite info
            char spriteInfo[64];
            snprintf(spriteInfo, sizeof(spriteInfo), "Sprite %zu/%zu", 
                    editorState.spriteData.currentSpriteIndex + 1,
                    editorState.spriteData.sprites.size());
            RenderText(spriteInfo);
            
            if (RenderButton("Next Sprite")) {
                if (editorState.spriteData.currentSpriteIndex < editorState.spriteData.sprites.size() - 1) {
                    editorState.spriteData.currentSpriteIndex++;
                }
            }
            
            // Zoom controls
            if (RenderButton("Zoom In")) {
                spriteViewerData.zoom = fminf(spriteViewerData.zoom * 1.25f, 8.0f);
            }
            
            if (RenderButton("Zoom Out")) {
                spriteViewerData.zoom = fmaxf(spriteViewerData.zoom * 0.8f, 0.125f);
            }
            
            if (RenderButton("Reset View")) {
                spriteViewerData.zoom = 1.0f;
                spriteViewerData.panOffset = (Clay_Vector2){0, 0};
            }
        }
    }
}
```

### VSE Editor Tab

```c
void RenderVSEEditorTab() {
    CLAY(CLAY_ID("VSEEditorTab"), 
         CLAY_LAYOUT(.direction = CLAY_TOP_TO_BOTTOM, .sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_GROW()})) {
        
        // VSE entry selector
        CLAY(CLAY_ID("VSESelector"), 
             CLAY_LAYOUT(.direction = CLAY_LEFT_TO_RIGHT, .sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_FIXED(40)})) {
            
            RenderText("VSE Entry:");
            
            // Entry dropdown/selector
            char entryLabel[32];
            snprintf(entryLabel, sizeof(entryLabel), "Entry %d (%d frames)", 
                    editorState.timeline.currentEntry,
                    editorState.vseData.tableEntries[editorState.timeline.currentEntry].pages);
            
            if (RenderDropdown(entryLabel, 256)) {
                // VSE entry selection logic
            }
            
            // Add/remove entry buttons
            if (RenderButton("Add Entry")) {
                // Add new VSE entry
            }
            
            if (RenderButton("Remove Entry")) {
                // Remove current VSE entry
            }
        }
        
        // Split view: Frame list + Frame editor
        CLAY(CLAY_ID("VSEContent"), 
             CLAY_LAYOUT(.direction = CLAY_LEFT_TO_RIGHT, .sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_GROW()})) {
            
            // Frame list (left panel)
            CLAY(CLAY_ID("FrameList"), 
                 CLAY_LAYOUT(.sizing = {CLAY_SIZING_FIXED(200), CLAY_SIZING_GROW()}),
                 CLAY_RECTANGLE(.color = {0.12f, 0.12f, 0.12f, 1.0f})) {
                
                VSETableEntry* entry = &editorState.vseData.tableEntries[editorState.timeline.currentEntry];
                
                for (int i = 0; i < entry->pages; i++) {
                    bool isSelected = (i == editorState.timeline.currentStep);
                    Clay_Color bgColor = isSelected ? 
                        (Clay_Color){0.3f, 0.5f, 0.8f, 1.0f} : 
                        (Clay_Color){0.15f, 0.15f, 0.15f, 1.0f};
                    
                    char frameId[32];
                    snprintf(frameId, sizeof(frameId), "Frame%d", i);
                    
                    CLAY(CLAY_ID(frameId), 
                         CLAY_LAYOUT(.sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_FIXED(30)}),
                         CLAY_RECTANGLE(.color = bgColor)) {
                        
                        VSEMove* move = GetVSEMove(&editorState.vseData, editorState.timeline.currentEntry, i);
                        
                        char frameLabel[64];
                        snprintf(frameLabel, sizeof(frameLabel), "Frame %d: Sprite %d (%d frames)", 
                                i, move->sprite, move->duration);
                        
                        if (RenderSelectableText(frameLabel, isSelected)) {
                            editorState.timeline.currentStep = i;
                        }
                    }
                }
            }
            
            // Frame editor (right panel)
            CLAY(CLAY_ID("FrameEditor"), 
                 CLAY_LAYOUT(.sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_GROW()})) {
                
                RenderVSEFrameEditor();
            }
        }
    }
}
```

### Timeline System Integration

```c
void RenderTimelinePanel() {
    CLAY(CLAY_ID("TimelinePanel"), 
         CLAY_LAYOUT(.direction = CLAY_TOP_TO_BOTTOM, .sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_FIXED(200)}),
         CLAY_RECTANGLE(.color = {0.08f, 0.08f, 0.08f, 1.0f})) {
        
        // Timeline controls
        CLAY(CLAY_ID("TimelineControls"), 
             CLAY_LAYOUT(.direction = CLAY_LEFT_TO_RIGHT, .sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_FIXED(40)})) {
            
            // Playback controls
            if (RenderButton(editorState.isPlaying ? "Pause" : "Play")) {
                editorState.isPlaying = !editorState.isPlaying;
            }
            
            if (RenderButton("Stop")) {
                editorState.isPlaying = false;
                editorState.playbackTime = 0.0f;
            }
            
            if (RenderButton("Step Forward")) {
                StepTimelineForward(&editorState);
            }
            
            if (RenderButton("Step Backward")) {
                StepTimelineBackward(&editorState);
            }
            
            // Timeline zoom
            RenderText("Zoom:");
            RenderSlider(&timelineData.zoomLevel, 0.1f, 5.0f);
            
            // Frame rate display
            char fpsLabel[32];
            snprintf(fpsLabel, sizeof(fpsLabel), "%.1f FPS", 60.0f);
            RenderText(fpsLabel);
        }
        
        // Timeline tracks
        CLAY(CLAY_ID("TimelineTracks"), 
             CLAY_LAYOUT(.direction = CLAY_TOP_TO_BOTTOM, .sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_GROW()})) {
            
            // Sprite track
            CLAY(CLAY_ID("SpriteTrack"), 
                 CLAY_LAYOUT(.sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_FIXED(40)}),
                 CLAY_CUSTOM_ELEMENT(.customElementType = ML2_ELEMENT_TIMELINE_TRACK,
                                    .customElementData = &timelineTrackData)) {}
            
            // Hitbox track
            if (timelineData.showHitboxTrack) {
                CLAY(CLAY_ID("HitboxTrack"), 
                     CLAY_LAYOUT(.sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_FIXED(30)})) {
                    RenderHitboxTimelineTrack();
                }
            }
            
            // Audio track
            if (timelineData.showAudioTrack) {
                CLAY(CLAY_ID("AudioTrack"), 
                     CLAY_LAYOUT(.sizing = {CLAY_SIZING_GROW(), CLAY_SIZING_FIXED(30)})) {
                    RenderAudioTimelineTrack();
                }
            }
        }
    }
}
```

## Input Handling

### Mouse and Keyboard Integration

```c
void HandleEditorInput() {
    // Get input from SDL3
    SDL_Event event;
    while (SDL_PollEvent(&event)) {
        switch (event.type) {
            case SDL_EVENT_MOUSE_WHEEL:
                if (IsMouseOverSpriteViewer()) {
                    // Zoom sprite viewer
                    spriteViewerData.zoom *= (event.wheel.y > 0) ? 1.1f : 0.9f;
                    spriteViewerData.zoom = Clamp(spriteViewerData.zoom, 0.1f, 10.0f);
                } else if (IsMouseOverTimeline()) {
                    // Zoom timeline
                    timelineData.zoomLevel *= (event.wheel.y > 0) ? 1.1f : 0.9f;
                    timelineData.zoomLevel = Clamp(timelineData.zoomLevel, 0.1f, 5.0f);
                }
                break;
                
            case SDL_EVENT_MOUSE_BUTTON_DOWN:
                if (event.button.button == SDL_BUTTON_MIDDLE) {
                    // Start panning
                    editorState.isPanning = true;
                    editorState.panStartPos = {event.button.x, event.button.y};
                } else if (IsMouseOverTimeline() && event.button.button == SDL_BUTTON_LEFT) {
                    // Seek timeline
                    float mouseX = event.button.x;
                    SeekTimelineToPosition(mouseX);
                }
                break;
                
            case SDL_EVENT_MOUSE_MOTION:
                if (editorState.isPanning) {
                    // Pan sprite viewer or timeline
                    float deltaX = event.motion.x - editorState.panStartPos.x;
                    float deltaY = event.motion.y - editorState.panStartPos.y;
                    
                    if (IsMouseOverSpriteViewer()) {
                        spriteViewerData.panOffset.x += deltaX;
                        spriteViewerData.panOffset.y += deltaY;
                    } else if (IsMouseOverTimeline()) {
                        timelineData.scrollOffset -= deltaX / (100.0f * timelineData.zoomLevel);
                    }
                    
                    editorState.panStartPos = {event.motion.x, event.motion.y};
                }
                break;
                
            case SDL_EVENT_KEY_DOWN:
                HandleKeyboardShortcuts(&event.key);
                break;
        }
        
        // Pass event to Clay for UI handling
        Clay_HandleInputEvent(&event);
    }
}

void HandleKeyboardShortcuts(SDL_KeyboardEvent* key) {
    switch (key->keysym.sym) {
        case SDLK_SPACE:
            // Toggle playback
            editorState.isPlaying = !editorState.isPlaying;
            break;
            
        case SDLK_LEFT:
            if (key->keysym.mod & KMOD_CTRL) {
                // Previous VSE entry
                if (editorState.timeline.currentEntry > 0) {
                    editorState.timeline.currentEntry--;
                    BuildTimelineFromVSE(&editorState.timeline);
                }
            } else {
                // Previous frame
                StepTimelineBackward(&editorState);
            }
            break;
            
        case SDLK_RIGHT:
            if (key->keysym.mod & KMOD_CTRL) {
                // Next VSE entry
                if (editorState.timeline.currentEntry < 255) {
                    editorState.timeline.currentEntry++;
                    BuildTimelineFromVSE(&editorState.timeline);
                }
            } else {
                // Next frame
                StepTimelineForward(&editorState);
            }
            break;
            
        case SDLK_TAB:
            // Cycle through tabs
            editorState.activeTab = (TabType)((editorState.activeTab + 1) % 6);
            break;
            
        case SDLK_s:
            if (key->keysym.mod & KMOD_CTRL) {
                // Save current data
                SaveCurrentData(&editorState);
            }
            break;
    }
}
```

## Data Flow and State Management

```c
typedef struct {
    // Core data
    CharacterInfo* currentCharacter;
    SpriteData spriteData;
    PaletteData paletteData;
    FRMData frmData;
    VSEData vseData;
    
    // UI state
    TabType activeTab;
    ML2_CustomElementData spriteViewerData;
    ML2_CustomElementData timelineTrackData;
    
    // Timeline state
    VSETimelineMapping timeline;
    float playbackTime;
    bool isPlaying;
    
    // Input state
    bool isPanning;
    Clay_Vector2 panStartPos;
    
    // File paths
    char basePath[256];
    char characterPath[256];
    
} EditorState;

// Global state management
EditorState g_editorState = {0};

void InitializeEditor(const char* basePath) {
    // Set up file paths
    strncpy(g_editorState.basePath, basePath, sizeof(g_editorState.basePath));
    
    // Load default character (Hikaru)
    LoadCharacter(&characters[0]);
    
    // Initialize timeline
    g_editorState.timeline.vseData = &g_editorState.vseData;
    g_editorState.timeline.currentEntry = 0;
    g_editorState.timeline.currentStep = 0;
    
    // Set default UI state
    g_editorState.activeTab = TAB_SPRITE_VIEWER;
    g_editorState.spriteViewerData.type = ML2_ELEMENT_SPRITE_VIEWER;
    g_editorState.spriteViewerData.spriteViewer.zoom = 1.0f;
}

void LoadCharacter(const CharacterInfo* character) {
    g_editorState.currentCharacter = (CharacterInfo*)character;
    
    // Build file paths
    char spritePath[512], palettePath[512], frmPath[512], vsePath[512];
    snprintf(spritePath, sizeof(spritePath), "%s/CHAR_SP/%s.SP", g_editorState.basePath, character->filePrefix);
    snprintf(palettePath, sizeof(palettePath), "%s/CHAR_PAL/%s.PAL", g_editorState.basePath, character->filePrefix);
    snprintf(frmPath, sizeof(frmPath), "%s/CHAR_FRM/%s.FRM", g_editorState.basePath, character->filePrefix);
    snprintf(vsePath, sizeof(vsePath), "%s/CHAR_VSE/%s.VSE", g_editorState.basePath, character->filePrefix);
    
    // Load data using existing functions
    LoadSpriteFile(spritePath, &g_editorState.spriteData);
    LoadPaletteFile(palettePath, &g_editorState.paletteData);
    LoadFRMFile(frmPath, &g_editorState.frmData);
    LoadVSEFile(vsePath, &g_editorState.vseData);
    
    // Rebuild timeline
    BuildTimelineFromVSE(&g_editorState.timeline);
    
    // Update custom element data
    g_editorState.spriteViewerData.spriteViewer.spriteData = &g_editorState.spriteData;
    g_editorState.spriteViewerData.spriteViewer.paletteData = &g_editorState.paletteData;
}
```

This design provides a comprehensive foundation for integrating Clay UI with your existing ML2 systems, creating a professional editor that leverages all your existing file format knowledge and rendering capabilities.