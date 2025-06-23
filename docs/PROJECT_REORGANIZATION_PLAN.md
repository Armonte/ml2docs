# Project Reorganization Plan

## ? Current Issues

### ? **What's Already Good**
- **Documentation structure** (`/docs`) is excellent and well-organized
- **Subdirectories** like `hooks/`, `menu/`, `engine/`, `ctx/` are properly structured
- **Third-party libraries** are properly contained in `third_party/`, `vendored/`

### ? **Main Problems** 
- **106+ loose files in root directory** - makes navigation impossible
- **Mixed file types** - source code, build artifacts, documentation, assets all together
- **Scattered documentation** - some docs in root, some in `/docs`
- **No clear source code organization** - main logic files scattered everywhere

## ?? **Proposed New Structure**

```
mlfixtest/
������ ? src/                          # ALL source code goes here
��   ������ ? core/                     # Core game systems
��   ��   ������ battleloop.cpp           # Main game loop (63KB)
��   ��   ������ hook.cpp                 # Main hook system (70KB) 
��   ��   ������ maingameloop_hook.cpp    # Game loop hooks
��   ��   ������ launcher.cpp             # DLL injection system
��   ��   ������ memory_mapper.cpp/.h     # Memory management
��   ��   ������ game_variables.cpp/.h    # Game state variables
��   ��   ������ game_memory.cpp/.h       # Memory definitions
��   ��   ������ address_definitions.h    # Memory addresses
��   ������ ? graphics/                 # Graphics and rendering
��   ��   ������ addFrmSpriteToRenderBuffer.cpp  # Sprite rendering
��   ��   ������ display_font_sprite_hook_*.* # Font rendering
��   ��   ������ fullscreen_crash_fix.s   # Graphics fixes
��   ������ ? input/                    # Input processing
��   ��   ������ sdl3_input_system.cpp/.hpp     # Main input system (71KB)
��   ��   ������ input_system_facade.cpp/.hpp  # Input facade
��   ��   ������ simple_input_hooks.cpp/.h     # Simple input hooks
��   ��   ������ input_serializer.cpp/.hpp     # Input serialization
��   ��   ������ decomp_handleP*input.c         # Decompiled input handlers
��   ��   ������ example-joystick*.c            # Input examples
��   ������ ? network/                  # Networking and netplay
��   ��   ������ netplay.cpp/.hpp         # Netplay system
��   ��   ������ gekkonet_integration.cpp # GekkoNet integration
��   ��   ������ network.hpp              # Network definitions
��   ��   ������ network_manager.hpp      # Network management
��   ��   ������ inputs_and_ack.hpp       # Network input sync
��   ������ ? savestate/                # Save states and recording
��   ��   ������ savestate.cpp/.h         # Save state system
��   ��   ������ savestate_manager.cpp/.hpp     # Save state management
��   ��   ������ recorder.cpp/.hpp        # Input recording
��   ��   ������ replayer.cpp/.hpp        # Replay system
��   ��   ������ ring_buffer_wrapper.hpp # Replay buffering
��   ������ ? timing/                   # Timing and synchronization
��   ��   ������ timing_system.cpp/.h     # Frame timing
��   ��   ������ game_loop_manager.cpp    # Game loop control
��   ������ ? decompiled/               # Decompiled game code
��   ��   ������ aeka.c                   # Decompiled functions
��   ��   ������ teambattle_snippet.c     # Team battle logic
��   ��   ������ createmainwindow.c       # Window creation
��   ������ ? utils/                    # Utility functions
��       ������ utilities.hpp            # General utilities  
��       ������ diagnostic.cpp           # Diagnostics
��       ������ debug_utils.cpp/.h       # Debug utilities
��
������ ? assets/                       # Game assets and data
��   ������ names.txt                    # Game data files
��   ������ global_variables_by_address_and_name.txt
��   ������ player.h                     # Player data structures  
��   ������ playerstruct.CSX            # Player struct definitions
��   ������ input_states.txt            # Input state data
��
������ ? build/                        # Build system and artifacts  
��   ������ ? scripts/                  # Build scripts
��   ��   ������ make_build               # Linux build script
��   ��   ������ make_build_mingw32       # MinGW32 build script
��   ��   ������ build_mingw32.bat        # Windows build batch
��   ��   ������ test_mingw32_env         # Environment test
��   ������ ? config/                   # Build configuration
��   ��   ������ launcher.ico             # App icon
��   ��   ������ launcher.rc              # Resource file
��   ��   ������ launcher-res.syso        # Resource object
��   ��   ������ launcher.exe.manifest    # Manifest
��   ��   ������ resources.rc.in          # Resource template
��   ������ ? final/                    # Final build outputs
��       ������ (existing final/ contents)
��
������ ? tools/                        # Development tools
��   ������ find_addresses.py           # Address discovery
��   ������ process_addresses.py        # Address processing  
��   ������ mcp-plugin.py               # MCP plugin
��   ������ pattern                     # Pattern matching
��
������ ? reference/                    # Documentation and reference
��   ������ SDL3_quickreference.txt     # SDL3 API reference
��   ������ SDL_quickref_*.txt          # SDL reference files
��   ������ sdl_quickref_gamepad_joystick_keyboard.txt
��   ������ SDL3_image_quickref.txt     # SDL image reference
��
������ ? assembly/                     # Assembly code and wrappers
��   ������ *.s files                   # All assembly files
��   ������ kuku.s, pupu.s              # Assembly snippets
��   ������ display_font_sprite_original.s
��
������ ? docs/                        # ? KEEP AS-IS (already perfect)
��   ������ README.md                   # Master documentation index
��   ������ architecture/               # System architecture
��   ������ development/                # Development guides
��   ������ fixes/                      # Bug fixes and solutions
��
������ ? hooks/                       # ? KEEP AS-IS (already organized)
������ ? menu/                        # ? KEEP AS-IS (already organized)
������ ? ctx/                         # ? KEEP AS-IS (already organized)
������ ? engine/                      # ? KEEP AS-IS (already organized)
������ ? caster_lib/                  # ? KEEP AS-IS (already organized)
������ ? third_party/                 # ? KEEP AS-IS (already organized)
������ ? vendored/                    # ? KEEP AS-IS (already organized)
��
������ CMakeLists.txt                  # ? KEEP IN ROOT
������ .gitignore                      # ? KEEP IN ROOT
������ mlfixtest.code-workspace        # ? KEEP IN ROOT
������ argentum.hpp                    # ? KEEP IN ROOT (main header)
������ README.md                       # ? CREATE PROJECT README
```

## ? **File Migration Plan**

### **Phase 1: Create New Directory Structure**
```bash
mkdir -p src/{core,graphics,input,network,savestate,timing,decompiled,utils}
mkdir -p assets
mkdir -p build/{scripts,config}
mkdir -p tools
mkdir -p reference  
mkdir -p assembly
```

### **Phase 2: Move Core System Files**
```bash
# Core game systems
mv battleloop.cpp src/core/
mv hook.cpp src/core/  
mv maingameloop_hook.cpp src/core/
mv launcher.cpp src/core/
mv memory_mapper.* src/core/
mv game_variables.* src/core/
mv game_memory.* src/core/
mv address_definitions.h src/core/
```

### **Phase 3: Move Graphics Files**
```bash
mv addFrmSpriteToRenderBuffer.cpp src/graphics/
mv display_font_sprite_hook_* src/graphics/
mv fullscreen_crash_fix.s src/graphics/
```

### **Phase 4: Move Input System Files**  
```bash
mv sdl3_input_system.* src/input/
mv input_system_facade.* src/input/
mv simple_input_hooks.* src/input/
mv input_serializer.* src/input/
mv decomp_handleP*input.c src/input/
mv example-joystick*.c src/input/
```

### **Phase 5: Move Network Files**
```bash
mv netplay.* src/network/
mv gekkonet_integration.cpp src/network/
mv network*.hpp src/network/
mv inputs_and_ack.hpp src/network/
```

### **Phase 6: Move Savestate/Recording Files**
```bash
mv savestate* src/savestate/
mv recorder.* src/savestate/
mv replayer.* src/savestate/
mv ring_buffer_wrapper.hpp src/savestate/
```

### **Phase 7: Move Timing Files**
```bash
mv timing_system.* src/timing/
mv game_loop_manager.cpp src/timing/
```

### **Phase 8: Move Decompiled Code**
```bash
mv aeka.c src/decompiled/
mv teambattle_snippet.c src/decompiled/
mv createmainwindow.c src/decompiled/
```

### **Phase 9: Move Utilities**
```bash
mv utilities.hpp src/utils/
mv diagnostic.cpp src/utils/
mv debug_utils.* src/utils/
```

### **Phase 10: Move Assets & Data**
```bash
mv names.txt assets/
mv global_variables_by_address_and_name.txt assets/
mv player.h assets/
mv playerstruct.CSX assets/
mv input_states.txt assets/
```

### **Phase 11: Move Build System**
```bash
mv make_build* build/scripts/
mv build_mingw32.bat build/scripts/
mv test_mingw32_env build/scripts/
mv launcher.ico build/config/
mv launcher.rc build/config/
mv launcher-res.syso build/config/
mv launcher.exe.manifest build/config/
mv resources.rc.in build/config/
```

### **Phase 12: Move Tools**
```bash
mv find_addresses.py tools/
mv process_addresses.py tools/
mv mcp-plugin.py tools/
mv pattern tools/
```

### **Phase 13: Move Reference Documentation**
```bash
mv SDL3_quickreference.txt reference/
mv SDL_quickref_*.txt reference/
mv sdl_quickref_gamepad_joystick_keyboard.txt reference/
mv SDL3_image_quickref.txt reference/
```

### **Phase 14: Move Assembly Files**
```bash
mv *.s assembly/
```

### **Phase 15: Clean Up Documentation**
```bash
# Move scattered docs to /docs if not already there
mv input_system.md docs/architecture/ # (if exists in root)
mv gekkonet_integration.md docs/architecture/networking.md # (if exists in root)
```

## ? **Required Updates After Reorganization**

### **1. Update CMakeLists.txt**
- Update all file paths to reflect new `src/` structure
- Update include directories
- Update target source files

### **2. Update Include Paths**
- Update `#include` statements to reflect new paths
- Use relative paths from `src/` directory
- Update header guards if needed

### **3. Update Build Scripts**
- Update paths in `build/scripts/make_build*`
- Update resource file paths
- Update output directories

### **4. Update VS Code Workspace**
- Update `mlfixtest.code-workspace` folder paths
- Update file associations
- Update build task paths

## ? **Benefits of This Organization**

### **? Clear Separation of Concerns**
- **Source code** (`src/`) separate from **build artifacts** (`build/`)
- **Game assets** (`assets/`) separate from **reference docs** (`reference/`)
- **Development tools** (`tools/`) clearly identified

### **? Easy Navigation**  
- Need input code? Look in `src/input/`
- Need graphics code? Look in `src/graphics/`
- Need to build? Look in `build/scripts/`
- Need documentation? Look in `docs/` (already perfect!)

### **? Better Team Development**
- Multiple developers can work on different `src/` subdirectories
- Clear ownership of different systems
- Easier code reviews

### **? Improved Build System**
- All build artifacts in `build/`
- Clean separation of source and generated files
- Easier to add new build targets

### **? Maintainable Codebase**
- Logical grouping of related functionality
- Easier to find and fix bugs
- Clear system boundaries

## ? **Implementation Notes**

### **Do This Carefully**
1. **Make a backup** before starting any moves
2. **Test builds** after each phase  
3. **Update one system at a time** (start with smallest)
4. **Update CMakeLists.txt incrementally** as you move files
5. **Keep your excellent `/docs` structure unchanged**

### **Start With**
- **Phase 14 (Assembly files)** - safest, least dependencies
- **Phase 10 (Assets)** - also safe, mostly data files
- **Phase 12 (Tools)** - Python scripts, independent

### **Save For Last**
- **Phase 2 (Core systems)** - most dependencies
- **Phase 4 (Input system)** - complex include relationships

Your `/docs` directory structure is actually **excellent** and doesn't need any changes - keep it exactly as it is! The main problem is just the scattered root directory files.