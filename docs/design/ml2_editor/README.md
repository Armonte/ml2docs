# ML2 Editor Design Documentation

This folder contains the user interface design and user experience documentation for the Moon Lights 2 Professional Animation Editor.

## ? **Documents Overview**

### ? **[Editor Design](editor_design.md)**
Overall editor design and architecture:
- Clay UI integration approach
- Tab-based workspace system  
- Professional IDE-like layout
- Tool integration strategy
- Cross-tool data sharing
- Implementation phases

### ?? **[UI Integration](ui_integration.md)**  
Detailed Clay UI implementation design:
- Custom element system for ML2 data types
- Specific tab implementations (Sprite Viewer, VSE Editor, etc.)
- Timeline interface design
- Input handling and keyboard shortcuts
- Data flow and state management
- Performance considerations

### ?? **[Timeline System Design](timeline_system.md)**
Professional timeline and state machine editor:
- Multi-track timeline architecture
- Frame-accurate playback system
- VSE integration with dual data systems
- State machine visual editor
- Animation player engine
- Advanced timeline features (keyframes, interpolation)

## ? **Design Philosophy**

### **Professional Tool Standards**
The ML2 Editor is designed to meet professional game development standards:
- **Familiar Interface**: IDE-like layout similar to Unity, Unreal, or Adobe tools
- **Efficient Workflow**: Minimal clicks to accomplish common tasks
- **Visual Clarity**: Clear visual hierarchy and information display
- **Responsive Design**: Smooth 60 FPS interaction even with large datasets

### **Fighting Game Specific**
Designed specifically for fighting game animation needs:
- **Frame-Accurate Editing**: Precise control over animation timing
- **Hitbox Visualization**: Clear display of collision data over sprites
- **State Machine Logic**: Visual editing of character state transitions
- **Dual Data Editing**: Separate but synchronized pattern and movement editing

## ?? **Key Design Decisions**

### **Clay UI Choice**
- **Modern Rendering**: Hardware-accelerated UI with custom elements
- **Flexible Layout**: Responsive layout system for complex interfaces
- **Custom Integration**: Perfect integration with ML2's specific data types
- **Performance**: Efficient rendering of timeline and animation data

### **Tabbed Interface**
Seven specialized tabs for different editing modes:
1. **Sprite Viewer** - Browse and view character sprites
2. **Hitbox Editor** - Edit collision boxes overlaid on sprites  
3. **VSE Editor** - Frame-by-frame VSE pattern editing
4. **Movement Editor** - Dedicated velocity/position editing with preview
5. **Animation Preview** - Real-time animation playback
6. **Palette Editor** - Color palette editing and effects
7. **State Machine** - Visual state transition editor

### **Timeline Integration**
Multi-track timeline supporting:
- **Sprite Track**: VSE pattern data with duration visualization
- **Movement Track**: Velocity data with position preview
- **Hitbox Track**: Attack/hurt/collision box visualization  
- **Audio Track**: Sound effect cues and timing
- **Control Flow Track**: Animation jumps and loop visualization

## ? **User Experience Goals**

### **Accessibility**
- **Keyboard Navigation**: Full keyboard accessibility
- **Screen Reader Support**: Proper accessibility annotations
- **Color Blind Friendly**: Alternative visual indicators beyond color
- **Scalable Interface**: Support for different screen sizes and DPI

### **Workflow Efficiency**
- **Undo/Redo**: Comprehensive command history with grouping
- **Copy/Paste**: Support for copying data between frames/entries
- **Batch Operations**: Multi-selection and bulk editing capabilities
- **Search & Filter**: Find specific frames, sprites, or properties quickly

### **Professional Features**
- **Project Management**: Save/load complete projects with all settings
- **Auto-Save**: Automatic backup to prevent data loss
- **Plugin Support**: Extensible architecture for custom tools
- **Help System**: Context-sensitive help and documentation

## ? **Implementation Highlights**

### **Custom Clay Elements**
Specialized UI elements for ML2 data:
```c
// Examples of custom elements
CLAY_ELEMENT_SPRITE_VIEWER         // Sprite display with zoom/pan
CLAY_ELEMENT_HITBOX_OVERLAY        // Hitbox visualization  
CLAY_ELEMENT_TIMELINE_TRACK        // Multi-track timeline
CLAY_ELEMENT_VSE_FRAME_EDITOR      // VSE frame property editor
CLAY_ELEMENT_MOVEMENT_PREVIEW      // Movement visualization
```

### **Professional Layout System**
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
„  Movement Track ????????????????????????????????????????     „ 
„  Hitbox Track   ????????????????????????????????????????     „ 
„¤„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„Ÿ„£
```

## ? **Design Documentation Quality**

- ? **Complete UI Specification** - Every interface element documented
- ? **Implementation Ready** - Specific Clay UI code examples
- ? **User Experience Focus** - Professional workflow considerations
- ? **Fighting Game Specific** - Tailored for animation editing needs
- ? **Accessibility Standards** - Modern accessibility requirements

## ? **Next Steps**

1. **Review**: [Editor Design](editor_design.md) for overall approach
2. **Implementation**: [UI Integration](ui_integration.md) for Clay UI specifics  
3. **Timeline**: [Timeline System Design](timeline_system.md) for advanced features
4. **Development**: [../../development/ml2_editor/implementation_roadmap.md](../../development/ml2_editor/implementation_roadmap.md) for implementation phases

---

**? Ready to create a beautiful and powerful animation editor interface!** ??