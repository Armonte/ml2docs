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
„¥„Ÿ„Ÿ ? src/                          # ALL source code goes here
„    „¥„Ÿ„Ÿ ? core/                     # Core game systems
„    „    „¥„Ÿ„Ÿ battleloop.cpp           # Main game loop (63KB)
„    „    „¥„Ÿ„Ÿ hook.cpp                 # Main hook system (70KB) 
„    „    „¥„Ÿ„Ÿ maingameloop_hook.cpp    # Game loop hooks
„    „    „¥„Ÿ„Ÿ launcher.cpp             # DLL injection system
„    „    „¥„Ÿ„Ÿ memory_mapper.cpp/.h     # Memory management
„    „    „¥„Ÿ„Ÿ game_variables.cpp/.h    # Game state variables
„    „    „¥„Ÿ„Ÿ game_memory.cpp/.h       # Memory definitions
„    „    „¤„Ÿ„Ÿ address_definitions.h    # Memory addresses
„    „¥„Ÿ„Ÿ ? graphics/                 # Graphics and rendering
„    „    „¥„Ÿ„Ÿ addFrmSpriteToRenderBuffer.cpp  # Sprite rendering
„    „    „¥„Ÿ„Ÿ display_font_sprite_hook_*.* # Font rendering
„    „    „¤„Ÿ„Ÿ fullscreen_crash_fix.s   # Graphics fixes
„    „¥„Ÿ„Ÿ ? input/                    # Input processing
„    „    „¥„Ÿ„Ÿ sdl3_input_system.cpp/.hpp     # Main input system (71KB)
„    „    „¥„Ÿ„Ÿ input_system_facade.cpp/.hpp  # Input facade
„    „    „¥„Ÿ„Ÿ simple_input_hooks.cpp/.h     # Simple input hooks
„    „    „¥„Ÿ„Ÿ input_serializer.cpp/.hpp     # Input serialization
„    „    „¥„Ÿ„Ÿ decomp_handleP*input.c         # Decompiled input handlers
„    „    „¤„Ÿ„Ÿ example-joystick*.c            # Input examples
„    „¥„Ÿ„Ÿ ? network/                  # Networking and netplay
„    „    „¥„Ÿ„Ÿ netplay.cpp/.hpp         # Netplay system
„    „    „¥„Ÿ„Ÿ gekkonet_integration.cpp # GekkoNet integration
„    „    „¥„Ÿ„Ÿ network.hpp              # Network definitions
„    „    „¥„Ÿ„Ÿ network_manager.hpp      # Network management
„    „    „¤„Ÿ„Ÿ inputs_and_ack.hpp       # Network input sync
„    „¥„Ÿ„Ÿ ? savestate/                # Save states and recording
„    „    „¥„Ÿ„Ÿ savestate.cpp/.h         # Save state system
„    „    „¥„Ÿ„Ÿ savestate_manager.cpp/.hpp     # Save state management
„    „    „¥„Ÿ„Ÿ recorder.cpp/.hpp        # Input recording
„    „    „¥„Ÿ„Ÿ replayer.cpp/.hpp        # Replay system
„    „    „¤„Ÿ„Ÿ ring_buffer_wrapper.hpp # Replay buffering
„    „¥„Ÿ„Ÿ ? timing/                   # Timing and synchronization
„    „    „¥„Ÿ„Ÿ timing_system.cpp/.h     # Frame timing
„    „    „¤„Ÿ„Ÿ game_loop_manager.cpp    # Game loop control
„    „¥„Ÿ„Ÿ ? decompiled/               # Decompiled game code
„    „    „¥„Ÿ„Ÿ aeka.c                   # Decompiled functions
„    „    „¥„Ÿ„Ÿ teambattle_snippet.c     # Team battle logic
„    „    „¤„Ÿ„Ÿ createmainwindow.c       # Window creation
„    „¤„Ÿ„Ÿ ? utils/                    # Utility functions
„        „¥„Ÿ„Ÿ utilities.hpp            # General utilities  
„        „¥„Ÿ„Ÿ diagnostic.cpp           # Diagnostics
„        „¤„Ÿ„Ÿ debug_utils.cpp/.h       # Debug utilities
„ 
„¥„Ÿ„Ÿ ? assets/                       # Game assets and data
„    „¥„Ÿ„Ÿ names.txt                    # Game data files
„    „¥„Ÿ„Ÿ global_variables_by_address_and_name.txt
„    „¥„Ÿ„Ÿ player.h                     # Player data structures  
„    „¥„Ÿ„Ÿ playerstruct.CSX            # Player struct definitions
„    „¤„Ÿ„Ÿ input_states.txt            # Input state data
„ 
„¥„Ÿ„Ÿ ? build/                        # Build system and artifacts  
„    „¥„Ÿ„Ÿ ? scripts/                  # Build scripts
„    „    „¥„Ÿ„Ÿ make_build               # Linux build script
„    „    „¥„Ÿ„Ÿ make_build_mingw32       # MinGW32 build script
„    „    „¥„Ÿ„Ÿ build_mingw32.bat        # Windows build batch
„    „    „¤„Ÿ„Ÿ test_mingw32_env         # Environment test
„    „¥„Ÿ„Ÿ ? config/                   # Build configuration
„    „    „¥„Ÿ„Ÿ launcher.ico             # App icon
„    „    „¥„Ÿ„Ÿ launcher.rc              # Resource file
„    „    „¥„Ÿ„Ÿ launcher-res.syso        # Resource object
„    „    „¥„Ÿ„Ÿ launcher.exe.manifest    # Manifest
„    „    „¤„Ÿ„Ÿ resources.rc.in          # Resource template
„    „¤„Ÿ„Ÿ ? final/                    # Final build outputs
„        „¤„Ÿ„Ÿ (existing final/ contents)
„ 
„¥„Ÿ„Ÿ ? tools/                        # Development tools
„    „¥„Ÿ„Ÿ find_addresses.py           # Address discovery
„    „¥„Ÿ„Ÿ process_addresses.py        # Address processing  
„    „¥„Ÿ„Ÿ mcp-plugin.py               # MCP plugin
„    „¤„Ÿ„Ÿ pattern                     # Pattern matching
„ 
„¥„Ÿ„Ÿ ? reference/                    # Documentation and reference
„    „¥„Ÿ„Ÿ SDL3_quickreference.txt     # SDL3 API reference
„    „¥„Ÿ„Ÿ SDL_quickref_*.txt          # SDL reference files
„    „¥„Ÿ„Ÿ sdl_quickref_gamepad_joystick_keyboard.txt
„    „¤„Ÿ„Ÿ SDL3_image_quickref.txt     # SDL image reference
„ 
„¥„Ÿ„Ÿ ? assembly/                     # Assembly code and wrappers
„    „¥„Ÿ„Ÿ *.s files                   # All assembly files
„    „¥„Ÿ„Ÿ kuku.s, pupu.s              # Assembly snippets
„    „¤„Ÿ„Ÿ display_font_sprite_original.s
„ 
„¥„Ÿ„Ÿ ? docs/                        # ? KEEP AS-IS (already perfect)
„    „¥„Ÿ„Ÿ README.md                   # Master documentation index
„    „¥„Ÿ„Ÿ architecture/               # System architecture
„    „¥„Ÿ„Ÿ development/                # Development guides
„    „¤„Ÿ„Ÿ fixes/                      # Bug fixes and solutions
„ 
„¥„Ÿ„Ÿ ? hooks/                       # ? KEEP AS-IS (already organized)
„¥„Ÿ„Ÿ ? menu/                        # ? KEEP AS-IS (already organized)
„¥„Ÿ„Ÿ ? ctx/                         # ? KEEP AS-IS (already organized)
„¥„Ÿ„Ÿ ? engine/                      # ? KEEP AS-IS (already organized)
„¥„Ÿ„Ÿ ? caster_lib/                  # ? KEEP AS-IS (already organized)
„¥„Ÿ„Ÿ ? third_party/                 # ? KEEP AS-IS (already organized)
„¥„Ÿ„Ÿ ? vendored/                    # ? KEEP AS-IS (already organized)
„ 
„¥„Ÿ„Ÿ CMakeLists.txt                  # ? KEEP IN ROOT
„¥„Ÿ„Ÿ .gitignore                      # ? KEEP IN ROOT
„¥„Ÿ„Ÿ mlfixtest.code-workspace        # ? KEEP IN ROOT
„¥„Ÿ„Ÿ argentum.hpp                    # ? KEEP IN ROOT (main header)
„¤„Ÿ„Ÿ README.md                       # ? CREATE PROJECT README
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