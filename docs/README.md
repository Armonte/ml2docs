# Moon Lights 2 Documentation

This directory contains comprehensive documentation for the Moon Lights 2 project, including both the original game systems and the new professional editing tools.

## ?? **Documentation Structure**

### ??? **Architecture Documentation**
Core system designs, technical specifications, and architectural decisions.

#### ML2 Editor Architecture
- **[Master System Architecture](architecture/ml2_editor/master_architecture.md)** - ?? **MAIN REFERENCE** - Complete technical specification with professional naming conventions
- **[File Formats Technical Reference](architecture/ml2_editor/file_formats.md)** - Complete binary file format specifications (.SP, .PAL, .FRM, .VSE)
- **[System Overview](architecture/ml2_editor/system_overview.md)** - High-level overview of all editor systems and capabilities

#### Original Game Systems
- **[Graphics System](architecture/graphics_system.md)** - Original graphics and rendering architecture
- **[Input System](architecture/input_system.md)** - Input handling and controller systems
- **[Input Refactoring Plan](architecture/input_system_refactor_plan.md)** - Planned input system improvements
- **[Input Refactoring Summary](architecture/input_refactoring_summary.md)** - Summary of input system changes
- **[Menu System](architecture/menu_system.md)** - Menu and UI system architecture
- **[Networking](architecture/networking.md)** - Network architecture and protocols

### ?? **Design Documentation**
User interface designs, user experience planning, and visual system designs.

#### ML2 Editor Design
- **[Editor Design](design/ml2_editor/editor_design.md)** - Overall editor design and Clay UI integration
- **[UI Integration](design/ml2_editor/ui_integration.md)** - Detailed Clay UI implementation with custom elements
- **[Timeline System Design](design/ml2_editor/timeline_system.md)** - Professional timeline and state machine editor

### ?? **Development Documentation**
Implementation guides, build instructions, and development workflows.

#### ML2 Editor Development
- **[Implementation Roadmap](development/ml2_editor/implementation_roadmap.md)** - Complete 8-phase development plan with milestones

#### General Development
- **[Build Instructions](development/build_instructions.md)** - How to build the project

### ?? **Fixes & Issues**
Bug fixes, issue tracking, and problem resolutions.

- **[UI Fixes](fixes/ui_fixes.md)** - User interface bug fixes and improvements

### ?? **Project Management**
- **[Organization Summary](ORGANIZATION_SUMMARY.md)** - Project organization and file structure

---

## ?? **Quick Start Guides**

### For Developers
1. **Start Here**: [Master System Architecture](architecture/ml2_editor/master_architecture.md)
2. **Understanding Data**: [File Formats Technical Reference](architecture/ml2_editor/file_formats.md)
3. **Implementation Plan**: [Implementation Roadmap](development/ml2_editor/implementation_roadmap.md)

### For Designers
1. **Editor Overview**: [Editor Design](design/ml2_editor/editor_design.md)
2. **UI Implementation**: [UI Integration](design/ml2_editor/ui_integration.md)
3. **Timeline Features**: [Timeline System Design](design/ml2_editor/timeline_system.md)

### For Project Managers
1. **System Overview**: [System Overview](architecture/ml2_editor/system_overview.md)
2. **Development Plan**: [Implementation Roadmap](development/ml2_editor/implementation_roadmap.md)
3. **Project Organization**: [Organization Summary](ORGANIZATION_SUMMARY.md)

---

## ?? **ML2 Editor: Professional Fighting Game Animation Editor**

The ML2 Editor represents a complete professional-grade fighting game animation editing suite designed to rival commercial tools like:

- **Guilty Gear Strive's** internal animation editor
- **Arc System Works'** animation pipeline
- **Street Fighter** development tools

### **Key Features**
- **Unified Interface**: All-in-one sprite, hitbox, and animation editing
- **Professional Timeline**: Multi-track timeline with frame-accurate editing
- **State Machine Editor**: Visual state transition editing
- **VSE System Integration**: Complete support for ML2's dual pattern/movement data
- **Clay UI Architecture**: Modern, responsive user interface
- **Project Management**: Complete project save/load with auto-backup
- **Undo/Redo System**: Professional command pattern implementation
- **Plugin Architecture**: Extensible system for custom tools

### **Technical Excellence**
- **Complete File Format Support**: Perfect compatibility with all ML2 formats
- **Memory Layout Precision**: Exact game memory integration
- **Performance Optimized**: 60 FPS editing with large datasets
- **Professional Testing**: Comprehensive unit and integration tests
- **Error Handling**: Robust validation and recovery systems
- **Accessibility**: International and accessibility standards support

---

## ?? **Documentation Standards**

### **File Naming Convention**
- Use lowercase with underscores: `system_architecture.md`
- Include category prefix when needed: `ml2_editor_design.md`
- Use descriptive names: `implementation_roadmap.md` not `roadmap.md`

### **Document Structure**
All documents should include:
1. **Clear title and overview**
2. **Table of contents for long documents**
3. **Code examples with proper syntax highlighting**
4. **Visual diagrams where helpful**
5. **Cross-references to related documents**

### **Code Documentation**
- Use proper C/C++ syntax highlighting
- Include complete struct definitions
- Provide function signatures and descriptions
- Add inline comments for complex logic

---

## ?? **Document Maintenance**

### **Version Control**
- All documentation is version controlled with the codebase
- Major changes should be reviewed like code changes
- Keep documentation in sync with implementation

### **Updates**
- Update documentation when systems change
- Cross-reference updates across related documents
- Maintain backward compatibility notes where needed

---

## ?? **Contributing to Documentation**

When adding new documentation:

1. **Choose the right location** based on the structure above
2. **Follow naming conventions** and formatting standards
3. **Update this README** to include links to new documents
4. **Cross-reference** related documents
5. **Include code examples** and technical details

---

**?? Ready to build the ultimate fighting game animation editor!** ???