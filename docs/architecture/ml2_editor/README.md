# ML2 Editor Architecture Documentation

This folder contains the core architectural documentation for the Moon Lights 2 Professional Animation Editor.

## ? **Documents Overview**

### ? **[Master System Architecture](master_architecture.md)** 
**? START HERE ?** - The definitive technical reference containing:
- Complete system architecture with professional naming conventions
- VSE system with dual pattern/movement data architecture  
- Entity system with proper slot management
- Player system with exact memory layout
- Collision box system with unified types
- Timeline and state machine architecture
- Clay UI integration design
- Project management, undo/redo, error handling
- Performance optimization and testing frameworks
- Plugin system and accessibility features

### ? **[File Formats Technical Reference](file_formats.md)**
Complete technical specifications for all ML2 binary formats:
- **Character System**: 14 characters with file organization
- **Palette System (.PAL)**: RGB555 format, 256 colors, conversion algorithms
- **Sprite System (.SP)**: 4-bit indexed color, tile-based storage
- **Frame System (.FRM)**: Big-endian pattern composition
- **VSE System (.VSE)**: Dual data architecture (pattern + movement)
- **Memory Integration**: Exact game memory addresses and layouts

### ? **[System Overview](system_overview.md)**
High-level overview and feature summary:
- Key technical achievements
- Fighting game editor capabilities  
- Professional tool comparison
- Implementation complexity overview
- Cross-system integration points

## ? **Quick Navigation**

### **For Developers**
1. **Start**: [Master System Architecture](master_architecture.md) - Complete technical spec
2. **Data Formats**: [File Formats](file_formats.md) - Binary format details
3. **Implementation**: [../../development/ml2_editor/implementation_roadmap.md](../../development/ml2_editor/implementation_roadmap.md) - Development phases

### **For Technical Leads**
1. **Architecture**: [Master System Architecture](master_architecture.md) - System design
2. **Complexity**: [System Overview](system_overview.md) - Scope understanding
3. **Planning**: [../../development/ml2_editor/implementation_roadmap.md](../../development/ml2_editor/implementation_roadmap.md) - Timeline and milestones

## ? **Key Technical Insights**

### **VSE System Discovery**
The most critical discovery was understanding ML2's **dual VSE system**:
- **Pattern Data**: Sprites, hitboxes, combat properties (48 bytes)
- **Movement Data**: Velocity, position changes (2 bytes) 
- **Separate File Storage**: Different offset pointers in VSE table
- **Control Flow**: Movement data includes animation jumps/loops

### **Memory Layout Precision**
- **Player Structure**: 284 bytes (0x11C) with exact field mapping
- **Entity System**: 8 slots Å~ 284 bytes per player
- **VSE Integration**: Direct mapping between VSE data and player state
- **Game Compatibility**: Byte-perfect memory layout preservation

### **Professional Architecture**
- **Naming Standards**: Consistent `System_VerbObject()` patterns
- **Error Handling**: Comprehensive error codes and recovery
- **Performance**: Memory pools, profiling, optimization strategies
- **Extensibility**: Plugin architecture and modular design

## ? **What This Enables**

### **Professional Fighting Game Editor**
This architecture supports building an editor that rivals:
- **Guilty Gear Strive's** internal animation tools
- **Arc System Works'** development pipeline
- **Street Fighter** development tools

### **Technical Capabilities**
- **Frame-accurate editing** with 60 FPS precision
- **Multi-track timeline** with pattern and movement data
- **Visual state machine** editor with transition logic
- **Real-time preview** with immediate feedback
- **Professional workflow** with undo/redo, project management
- **Plugin extensibility** for custom tools

## ? **Documentation Quality**

- ? **Complete Technical Specifications** - Every data structure documented
- ? **Professional Naming Conventions** - No more `unk_` fields  
- ? **Memory Layout Precision** - Exact byte-level compatibility
- ? **Cross-System Integration** - How all systems work together
- ? **Implementation Ready** - Ready for development team

---

**? Ready to build the ultimate fighting game animation editor!** ??