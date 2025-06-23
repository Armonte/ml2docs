# Moon Lights 2 Timeline & State Machine Editor Design

## Overview

This document details the timeline system and character state machine editor for the ML2 editing tool. The system will allow visual editing of character animations, move sequences, and state transitions with frame-accurate playback and duration control.

## Core Concepts

### Character State Machine Architecture

```
Character State Machine
„¥„Ÿ„Ÿ States (Idle, Walk, Attack, etc.)
„    „¥„Ÿ„Ÿ Animation Sequences (VSE Entries)
„    „    „¥„Ÿ„Ÿ Pages (Individual Frames)
„    „    „    „¥„Ÿ„Ÿ Duration (frames to hold)
„    „    „    „¥„Ÿ„Ÿ Sprite Data
„    „    „    „¥„Ÿ„Ÿ Hitbox Data
„    „    „    „¤„Ÿ„Ÿ Audio Cues
„    „    „¤„Ÿ„Ÿ Transition Rules
„    „¤„Ÿ„Ÿ State Metadata
„¥„Ÿ„Ÿ Global Properties
„    „¥„Ÿ„Ÿ Frame Rate (60 FPS standard)
„    „¥„Ÿ„Ÿ State Priorities
„    „¤„Ÿ„Ÿ Default Transitions
„¤„Ÿ„Ÿ Animation Curves
    „¥„Ÿ„Ÿ Position Interpolation
    „¥„Ÿ„Ÿ Rotation Curves
    „¤„Ÿ„Ÿ Scale Animations
```

### VSE Duration System

```c
typedef struct {
    uint8_t sprite;           // Sprite pattern index
    int8_t duration;          // Duration in frames (1-255)
    // ... hitbox data ...
    // ... other frame data ...
} VSEMove;

typedef struct {
    VSEMove* frames;          // Array of frames
    int totalFrames;          // Total number of frames
    int totalDuration;        // Sum of all frame durations
    bool isLooping;           // Whether animation loops
    int loopStartFrame;       // Frame to loop back to
} AnimationSequence;
```

## Timeline System Architecture

### Timeline Data Structure

```c
typedef struct {
    // Playback state
    float currentTime;        // Current playback time in seconds
    int currentFrame;         // Current frame index
    int currentPage;          // Current page within frame
    bool isPlaying;           // Playback state
    bool isPaused;            // Pause state
    float playbackSpeed;      // Speed multiplier (1.0 = normal)
    
    // Timeline properties
    float timelineStart;      // Start time of visible timeline
    float timelineEnd;        // End time of visible timeline
    float timelineZoom;       // Zoom level of timeline
    float frameDuration;      // Duration of one frame (1/60th second)
    
    // Selection and editing
    int selectedFrameStart;   // Start of selection
    int selectedFrameEnd;     // End of selection
    bool isSelecting;         // Currently selecting frames
    
    // Animation data
    AnimationSequence* currentSequence;
    CharacterState* currentState;
    
} TimelineState;

typedef struct {
    // Visual properties
    float trackHeight;        // Height of each track
    float rulerHeight;        // Height of time ruler
    Color backgroundColor;
    Color gridColor;
    Color playheadColor;
    Color selectionColor;
    
    // Timeline tracks
    TimelineTrack tracks[MAX_TIMELINE_TRACKS];
    int numTracks;
    
} TimelineProperties;
```

### Timeline Track System

```c
typedef enum {
    TRACK_TYPE_SPRITE,        // Sprite animation track
    TRACK_TYPE_HITBOX,        // Hitbox animation track
    TRACK_TYPE_HURTBOX,       // Hurtbox animation track
    TRACK_TYPE_AUDIO,         // Audio cue track
    TRACK_TYPE_EFFECT,        // Visual effect track
    TRACK_TYPE_MOVEMENT,      // Movement/position track
    TRACK_TYPE_STATE,         // State transition track
} TimelineTrackType;

typedef struct {
    TimelineTrackType type;
    char name[64];
    bool isVisible;
    bool isMuted;             // For audio tracks
    bool isLocked;            // Prevent editing
    Color trackColor;
    
    // Track data
    TimelineKeyframe* keyframes;
    int numKeyframes;
    
} TimelineTrack;

typedef struct {
    float time;               // Time position in seconds
    int frame;                // Frame number
    int duration;             // Duration in frames
    
    // Data based on track type
    union {
        SpriteKeyframe sprite;
        HitboxKeyframe hitbox;
        AudioKeyframe audio;
        EffectKeyframe effect;
        MovementKeyframe movement;
        StateKeyframe state;
    } data;
    
    // Visual properties
    bool isSelected;
    Color keyframeColor;
    
} TimelineKeyframe;
```

## Clay UI Timeline Interface

### Timeline Layout Structure

```c
void RenderTimelineTab() {
    CLAY({ .id = CLAY_ID("TimelineEditor"), .layout = { .layoutDirection = CLAY_TOP_TO_BOTTOM } }) {
        
        // Timeline toolbar
        CLAY({ .id = CLAY_ID("TimelineToolbar"), .layout = { .sizing = { .height = CLAY_SIZING_FIXED(40) } } }) {
            RenderTimelineToolbar();
        }
        
        // Main timeline area
        CLAY({ .id = CLAY_ID("TimelineMain"), .layout = { 
            .layoutDirection = CLAY_LEFT_TO_RIGHT, 
            .sizing = { .height = CLAY_SIZING_GROW() } 
        } }) {
            
            // Track list (left panel)
            CLAY({ .id = CLAY_ID("TrackList"), .layout = { .sizing = { .width = CLAY_SIZING_FIXED(200) } } }) {
                RenderTrackList();
            }
            
            // Timeline canvas
            CLAY({ .id = CLAY_ID("TimelineCanvas"), .layout = { .sizing = { .width = CLAY_SIZING_GROW() } } }) {
                // Custom timeline rendering
                TimelineRenderData timelineData = {
                    .state = &g_timelineState,
                    .properties = &g_timelineProperties
                };
                CLAY({ .id = CLAY_ID("TimelineViewer"), .custom = { .customData = &timelineData } }) {}
            }
        }
        
        // Timeline controls (bottom)
        CLAY({ .id = CLAY_ID("TimelineControls"), .layout = { .sizing = { .height = CLAY_SIZING_FIXED(60) } } }) {
            RenderTimelineControls();
        }
    }
}
```

### Timeline Toolbar

```c
void RenderTimelineToolbar() {
    CLAY({ .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT, .childGap = 8, .padding = CLAY_PADDING_ALL(8) } }) {
        
        // Playback controls
        CLAY({ .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT, .childGap = 4 } }) {
            RenderPlayButton();
            RenderPauseButton();
            RenderStopButton();
            RenderRecordButton();
        }
        
        // Separator
        CLAY({ .layout = { .sizing = { .width = CLAY_SIZING_FIXED(1), .height = CLAY_SIZING_FIXED(24) } },
               .backgroundColor = {128, 128, 128, 255} }) {}
        
        // Timeline tools
        CLAY({ .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT, .childGap = 4 } }) {
            RenderSelectTool();
            RenderMoveTool();
            RenderScaleTool();
            RenderCutTool();
        }
        
        // Separator
        CLAY({ .layout = { .sizing = { .width = CLAY_SIZING_FIXED(1), .height = CLAY_SIZING_FIXED(24) } },
               .backgroundColor = {128, 128, 128, 255} }) {}
        
        // Timeline properties
        CLAY({ .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT, .childGap = 8 } }) {
            CLAY_TEXT(CLAY_STRING("Frame Rate:"), CLAY_TEXT_CONFIG({ .textColor = {255, 255, 255, 255} }));
            RenderFrameRateSelector();
            
            CLAY_TEXT(CLAY_STRING("Snap:"), CLAY_TEXT_CONFIG({ .textColor = {255, 255, 255, 255} }));
            RenderSnapToggle();
        }
    }
}
```

### Timeline Controls

```c
void RenderTimelineControls() {
    CLAY({ .layout = { .layoutDirection = CLAY_TOP_TO_BOTTOM, .padding = CLAY_PADDING_ALL(8) } }) {
        
        // Time scrubber
        CLAY({ .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT, .childAlignment = { .y = CLAY_ALIGN_Y_CENTER } } }) {
            // Current time display
            char timeText[32];
            snprintf(timeText, sizeof(timeText), "%02d:%02d.%02d", 
                    (int)(g_timelineState.currentTime / 60),
                    (int)(g_timelineState.currentTime) % 60,
                    (int)((g_timelineState.currentTime - (int)g_timelineState.currentTime) * 100));
            CLAY_TEXT(CLAY_STRING(timeText), CLAY_TEXT_CONFIG({ .textColor = {255, 255, 255, 255} }));
            
            // Timeline scrubber
            CLAY({ .id = CLAY_ID("TimeScrubber"), .layout = { .sizing = { .width = CLAY_SIZING_GROW() } } }) {
                TimelineScrubberData scrubberData = { .timelineState = &g_timelineState };
                CLAY({ .custom = { .customData = &scrubberData } }) {}
            }
            
            // Total duration
            char durationText[32];
            float totalDuration = CalculateTotalDuration(g_timelineState.currentSequence);
            snprintf(durationText, sizeof(durationText), "%02d:%02d.%02d", 
                    (int)(totalDuration / 60),
                    (int)(totalDuration) % 60,
                    (int)((totalDuration - (int)totalDuration) * 100));
            CLAY_TEXT(CLAY_STRING(durationText), CLAY_TEXT_CONFIG({ .textColor = {200, 200, 200, 255} }));
        }
        
        // Zoom and frame controls
        CLAY({ .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT, .childGap = 16 } }) {
            // Zoom controls
            CLAY({ .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT, .childGap = 4 } }) {
                CLAY_TEXT(CLAY_STRING("Zoom:"), CLAY_TEXT_CONFIG({ .textColor = {255, 255, 255, 255} }));
                RenderZoomSlider();
                RenderFitToWindowButton();
            }
            
            // Frame navigation
            CLAY({ .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT, .childGap = 4 } }) {
                RenderPreviousFrameButton();
                char frameText[16];
                snprintf(frameText, sizeof(frameText), "Frame %d", g_timelineState.currentFrame);
                CLAY_TEXT(CLAY_STRING(frameText), CLAY_TEXT_CONFIG({ .textColor = {255, 255, 255, 255} }));
                RenderNextFrameButton();
            }
        }
    }
}
```

## State Machine Editor Integration

### State Graph Visualization

```c
typedef struct {
    char name[32];            // State name (e.g., "Idle", "Walk", "Attack")
    StateType type;           // NORMAL, SPECIAL, SUPER, etc.
    AnimationSequence* animation;
    
    // State properties
    int priority;             // State priority (higher = harder to interrupt)
    int startupFrames;        // Frames before active
    int activeFrames;         // Active frames
    int recoveryFrames;       // Recovery frames
    bool canCancel;           // Can be canceled into other moves
    
    // Visual position in state graph
    Clay_Vector2 graphPosition;
    bool isSelected;
    
    // Transitions
    StateTransition* transitions;
    int numTransitions;
    
} CharacterState;

typedef struct {
    CharacterState* fromState;
    CharacterState* toState;
    TransitionCondition condition;
    int transitionFrames;     // Frames to blend between states
    
} StateTransition;

typedef enum {
    TRANSITION_ON_INPUT,      // Triggered by input
    TRANSITION_ON_FRAME,      // Triggered at specific frame
    TRANSITION_ON_HIT,        // Triggered when hitting opponent
    TRANSITION_ON_BLOCK,      // Triggered when blocked
    TRANSITION_ON_WHIFF,      // Triggered when missing
    TRANSITION_AUTO,          // Automatic transition
} TransitionCondition;
```

### State Machine Clay Interface

```c
void RenderStateMachineTab() {
    CLAY({ .id = CLAY_ID("StateMachineEditor"), .layout = { .layoutDirection = CLAY_LEFT_TO_RIGHT } }) {
        
        // State list (left panel)
        CLAY({ .id = CLAY_ID("StateList"), .layout = { 
            .sizing = { .width = CLAY_SIZING_FIXED(250) },
            .layoutDirection = CLAY_TOP_TO_BOTTOM
        } }) {
            RenderStateList();
        }
        
        // Main editing area
        CLAY({ .id = CLAY_ID("StateMachineMain"), .layout = { 
            .layoutDirection = CLAY_TOP_TO_BOTTOM,
            .sizing = { .width = CLAY_SIZING_GROW() }
        } }) {
            
            // State graph view
            CLAY({ .id = CLAY_ID("StateGraph"), .layout = { .sizing = { .height = CLAY_SIZING_PERCENT(0.6f) } } }) {
                StateGraphRenderData graphData = { .stateMachine = &g_currentStateMachine };
                CLAY({ .custom = { .customData = &graphData } }) {}
            }
            
            // Timeline view for selected state
            CLAY({ .id = CLAY_ID("StateTimeline"), .layout = { .sizing = { .height = CLAY_SIZING_PERCENT(0.4f) } } }) {
                if (g_selectedState) {
                    RenderStateTimeline(g_selectedState);
                }
            }
        }
        
        // Properties panel (right)
        CLAY({ .id = CLAY_ID("StateProperties"), .layout = { .sizing = { .width = CLAY_SIZING_FIXED(300) } } }) {
            RenderStateProperties();
        }
    }
}
```

## Animation Playback System

### Playback Engine

```c
typedef struct {
    // Current playback state
    AnimationSequence* currentSequence;
    int currentFrameIndex;
    int currentPageIndex;
    float frameTimer;         // Time accumulated in current frame
    
    // Playback control
    bool isPlaying;
    bool isPaused;
    bool isLooping;
    float speed;              // Playback speed multiplier
    
    // Callbacks
    void (*onFrameChanged)(int newFrame, void* userData);
    void (*onAnimationComplete)(void* userData);
    void (*onHitboxActive)(HitboxData* hitbox, void* userData);
    void* callbackUserData;
    
} AnimationPlayer;

// Update the animation player (called each frame)
void UpdateAnimationPlayer(AnimationPlayer* player, float deltaTime) {
    if (!player->isPlaying || player->isPaused || !player->currentSequence) {
        return;
    }
    
    // Advance timer
    player->frameTimer += deltaTime * player->speed;
    
    // Get current VSE move
    VSEMove* currentMove = &player->currentSequence->frames[player->currentFrameIndex];
    float frameDuration = currentMove->duration * (1.0f / 60.0f); // Convert frames to seconds
    
    // Check if we should advance to next frame
    if (player->frameTimer >= frameDuration) {
        player->frameTimer -= frameDuration;
        
        // Advance frame
        player->currentFrameIndex++;
        
        // Check for loop or end
        if (player->currentFrameIndex >= player->currentSequence->totalFrames) {
            if (player->isLooping) {
                player->currentFrameIndex = player->currentSequence->loopStartFrame;
            } else {
                player->isPlaying = false;
                if (player->onAnimationComplete) {
                    player->onAnimationComplete(player->callbackUserData);
                }
                return;
            }
        }
        
        // Fire frame changed callback
        if (player->onFrameChanged) {
            player->onFrameChanged(player->currentFrameIndex, player->callbackUserData);
        }
        
        // Check for hitbox activation
        VSEMove* newMove = &player->currentSequence->frames[player->currentFrameIndex];
        if (newMove->hitbox1_Width > 0 || newMove->hitbox2_Width > 0) {
            if (player->onHitboxActive) {
                HitboxData hitboxData = ExtractHitboxData(newMove);
                player->onHitboxActive(&hitboxData, player->callbackUserData);
            }
        }
    }
}
```

### Timeline Scrubbing

```c
typedef struct {
    float position;           // Position along timeline (0.0 to 1.0)
    bool isDragging;          // Currently being dragged
    float snapThreshold;      // Distance for frame snapping
    
} TimelineScrubber;

void HandleTimelineScrubbing(TimelineScrubber* scrubber, TimelineState* timeline, Clay_Vector2 mousePos, bool mouseDown) {
    // Calculate timeline bounds
    Clay_BoundingBox timelineBounds = GetTimelineBounds();
    
    if (mouseDown && IsPointInBounds(mousePos, timelineBounds)) {
        scrubber->isDragging = true;
        
        // Calculate position along timeline
        float relativeX = (mousePos.x - timelineBounds.x) / timelineBounds.width;
        scrubber->position = ClampFloat(relativeX, 0.0f, 1.0f);
        
        // Convert to time
        float totalDuration = CalculateTotalDuration(timeline->currentSequence);
        timeline->currentTime = scrubber->position * totalDuration;
        
        // Snap to frames if enabled
        if (timeline->snapToFrames) {
            int frameIndex = (int)(timeline->currentTime * 60.0f); // 60 FPS
            timeline->currentTime = frameIndex / 60.0f;
            timeline->currentFrame = frameIndex;
        }
        
        // Pause playback while scrubbing
        timeline->isPlaying = false;
    } else if (!mouseDown) {
        scrubber->isDragging = false;
    }
}
```

## Advanced Timeline Features

### Keyframe Interpolation

```c
typedef enum {
    INTERPOLATION_STEP,       // No interpolation (discrete)
    INTERPOLATION_LINEAR,     // Linear interpolation
    INTERPOLATION_EASE_IN,    // Ease in curve
    INTERPOLATION_EASE_OUT,   // Ease out curve
    INTERPOLATION_EASE_IN_OUT,// Ease in-out curve
    INTERPOLATION_BEZIER,     // Custom bezier curve
} InterpolationType;

typedef struct {
    InterpolationType type;
    float tension;            // Curve tension (for ease modes)
    Clay_Vector2 controlPoint1; // For bezier curves
    Clay_Vector2 controlPoint2; // For bezier curves
} InterpolationCurve;

// Interpolate between two keyframes
float InterpolateKeyframes(TimelineKeyframe* keyframe1, TimelineKeyframe* keyframe2, float t, InterpolationCurve curve) {
    switch (curve.type) {
        case INTERPOLATION_STEP:
            return t < 1.0f ? keyframe1->data.value : keyframe2->data.value;
            
        case INTERPOLATION_LINEAR:
            return Lerp(keyframe1->data.value, keyframe2->data.value, t);
            
        case INTERPOLATION_EASE_IN:
            return EaseIn(keyframe1->data.value, keyframe2->data.value, t, curve.tension);
            
        case INTERPOLATION_EASE_OUT:
            return EaseOut(keyframe1->data.value, keyframe2->data.value, t, curve.tension);
            
        case INTERPOLATION_EASE_IN_OUT:
            return EaseInOut(keyframe1->data.value, keyframe2->data.value, t, curve.tension);
            
        case INTERPOLATION_BEZIER:
            return BezierInterpolate(keyframe1->data.value, keyframe2->data.value, t, curve.controlPoint1, curve.controlPoint2);
    }
}
```

### Multi-Track Editing

```c
typedef struct {
    TimelineTrack* tracks[MAX_TIMELINE_TRACKS];
    int numTracks;
    int selectedTrack;
    
    // Multi-selection
    bool multiSelectMode;
    bool selectedTracks[MAX_TIMELINE_TRACKS];
    
    // Group operations
    TimelineGroup* groups;
    int numGroups;
    
} MultiTrackEditor;

typedef struct {
    char name[64];
    int trackIndices[32];
    int numTracks;
    bool isCollapsed;
    Color groupColor;
} TimelineGroup;

// Group operations
void GroupSelectedTracks(MultiTrackEditor* editor, const char* groupName) {
    TimelineGroup newGroup = {0};
    strncpy(newGroup.name, groupName, sizeof(newGroup.name));
    
    for (int i = 0; i < MAX_TIMELINE_TRACKS; i++) {
        if (editor->selectedTracks[i]) {
            newGroup.trackIndices[newGroup.numTracks++] = i;
        }
    }
    
    editor->groups[editor->numGroups++] = newGroup;
}

void UngroupTracks(MultiTrackEditor* editor, int groupIndex) {
    // Remove group and shift remaining groups
    for (int i = groupIndex; i < editor->numGroups - 1; i++) {
        editor->groups[i] = editor->groups[i + 1];
    }
    editor->numGroups--;
}
```

## Integration with VSE Data

### VSE Timeline Mapping

```c
typedef struct {
    VSEData* vseData;         // Reference to VSE data
    int currentEntry;         // Current VSE table entry
    int currentStep;          // Current step within entry
    
    // Timeline mapping
    TimelineFrame* timelineFrames;
    int numTimelineFrames;
    
} VSETimelineMapping;

typedef struct {
    int vseEntryIndex;        // Which VSE entry this frame belongs to
    int vseStepIndex;         // Which step within the entry
    VSEMove* vseMove;         // Pointer to the actual VSE move data
    
    // Calculated timeline properties
    float startTime;          // Start time in timeline
    float duration;           // Duration in seconds
    int frameNumber;          // Frame number in timeline
    
} TimelineFrame;

// Convert VSE data to timeline format
void BuildTimelineFromVSE(VSETimelineMapping* mapping) {
    mapping->numTimelineFrames = 0;
    float currentTime = 0.0f;
    int frameNumber = 0;
    
    VSETableEntry* entry = &mapping->vseData->tableEntries[mapping->currentEntry];
    
    for (int step = 0; step < entry->pages; step++) {
        uint16_t fileOffset = ((entry->entryOffset_hi << 8) | entry->entryOffset_lo);
        size_t moveIndex = (fileOffset - 0x2820) / 0x30 + step;
        VSEMove* move = &mapping->vseData->moves[moveIndex];
        
        TimelineFrame frame = {0};
        frame.vseEntryIndex = mapping->currentEntry;
        frame.vseStepIndex = step;
        frame.vseMove = move;
        frame.startTime = currentTime;
        frame.duration = move->duration / 60.0f; // Convert frames to seconds
        frame.frameNumber = frameNumber++;
        
        mapping->timelineFrames[mapping->numTimelineFrames++] = frame;
        currentTime += frame.duration;
    }
}
```

## Export and Import System

### Timeline Data Export

```c
typedef struct {
    char name[64];
    char description[256];
    
    // Animation metadata
    float frameRate;
    int totalFrames;
    float totalDuration;
    bool isLooping;
    
    // Track data
    TimelineTrackExport tracks[MAX_TIMELINE_TRACKS];
    int numTracks;
    
    // State machine data
    CharacterStateExport states[MAX_STATES];
    int numStates;
    StateTransitionExport transitions[MAX_TRANSITIONS];
    int numTransitions;
    
} TimelineProjectExport;

// Export timeline to JSON
bool ExportTimelineProject(const char* filepath, TimelineProjectExport* project) {
    FILE* file = fopen(filepath, "w");
    if (!file) return false;
    
    fprintf(file, "{\n");
    fprintf(file, "  \"name\": \"%s\",\n", project->name);
    fprintf(file, "  \"frameRate\": %.2f,\n", project->frameRate);
    fprintf(file, "  \"totalDuration\": %.4f,\n", project->totalDuration);
    fprintf(file, "  \"isLooping\": %s,\n", project->isLooping ? "true" : "false");
    
    // Export tracks
    fprintf(file, "  \"tracks\": [\n");
    for (int i = 0; i < project->numTracks; i++) {
        ExportTimelineTrack(file, &project->tracks[i], i == project->numTracks - 1);
    }
    fprintf(file, "  ],\n");
    
    // Export states
    fprintf(file, "  \"states\": [\n");
    for (int i = 0; i < project->numStates; i++) {
        ExportCharacterState(file, &project->states[i], i == project->numStates - 1);
    }
    fprintf(file, "  ]\n");
    
    fprintf(file, "}\n");
    fclose(file);
    return true;
}
```

## Performance Considerations

### Efficient Timeline Rendering

```c
typedef struct {
    // Visible range optimization
    float visibleStartTime;
    float visibleEndTime;
    int visibleStartFrame;
    int visibleEndFrame;
    
    // LOD system
    float timelineZoom;
    TimelineLOD currentLOD;
    
    // Dirty regions
    bool needsRedraw;
    Clay_BoundingBox dirtyRegion;
    
} TimelineRenderOptimization;

typedef enum {
    TIMELINE_LOD_DETAILED,    // Show all keyframes and details
    TIMELINE_LOD_MEDIUM,      // Show major keyframes only
    TIMELINE_LOD_OVERVIEW,    // Show track blocks only
} TimelineLOD;

// Only render visible timeline elements
void RenderTimelineOptimized(TimelineRenderOptimization* opt) {
    // Determine LOD based on zoom level
    if (opt->timelineZoom > 10.0f) {
        opt->currentLOD = TIMELINE_LOD_DETAILED;
    } else if (opt->timelineZoom > 2.0f) {
        opt->currentLOD = TIMELINE_LOD_MEDIUM;
    } else {
        opt->currentLOD = TIMELINE_LOD_OVERVIEW;
    }
    
    // Only process visible keyframes
    for (int trackIndex = 0; trackIndex < g_timelineProperties.numTracks; trackIndex++) {
        TimelineTrack* track = &g_timelineProperties.tracks[trackIndex];
        
        for (int keyIndex = 0; keyIndex < track->numKeyframes; keyIndex++) {
            TimelineKeyframe* keyframe = &track->keyframes[keyIndex];
            
            // Skip if outside visible range
            if (keyframe->time < opt->visibleStartTime || keyframe->time > opt->visibleEndTime) {
                continue;
            }
            
            RenderKeyframe(keyframe, opt->currentLOD);
        }
    }
}
```

## Conclusion

This timeline and state machine system provides:

- **Professional Animation Tools**: Frame-accurate playback, scrubbing, and editing
- **Visual State Machine**: Graph-based state editing with transition visualization  
- **VSE Integration**: Direct editing of VSE frame data with duration control
- **Multi-Track Timeline**: Separate tracks for sprites, hitboxes, audio, etc.
- **Keyframe Animation**: Support for interpolated properties and curves
- **Export/Import**: Save and load complete animation projects

The system transforms your existing VSE data into a professional animation editing environment while maintaining compatibility with the original game data formats.