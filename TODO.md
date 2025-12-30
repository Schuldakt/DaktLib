# DaktLib TODO

## Module Implementation Status

| Module | Status | Headers | Implementation | Tests | Docs |
|--------|--------|---------|----------------|-------|------|
| Core | ✅ Complete | ✅ | ✅ | ⬜ | ✅ |
| Logger | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| Events | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| Config | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| VFS | ✅ Core Complete | ✅ | ✅ | ⬜ | ⬜ |
| Parser | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| GUI | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| Overlay | ⬜ Not Started | ⬜ | ⬜ | ⬜ | ⬜ |
| OCR | ⬜ Not Started | ⬜ | ⬜ | ⬜ | ⬜ |
| Export | ⬜ Not Started | ⬜ | ⬜ | ⬜ | ⬜ |

**Legend:** ✅ Complete | 🔨 In Progress | ⬜ Not Started

---

## Implementation Order

### Phase 1: Foundation ✅ COMPLETE
1. [x] **Core** - Types, Memory, String, Buffer, Hash, Time, FileSystem, Platform
2. [x] **Logger** - Multi-sink logging with levels and formatting
3. [x] **Events** - EventBus, Signal/Slot, queued events
4. [x] **Config** - INI/TOML/JSON parsing (built-in), hot-reload via ConfigWatcher

### Phase 2: Data Access ✅ COMPLETE
5. [x] **VFS** - Virtual file system with mount points (core complete, archive providers pending zlib)
6. [x] **Parser** - Star Citizen file format handlers (core parsers complete, crypto pending)

### Phase 3: UI & Tools ✅ PARTIAL
7. [x] **GUI** - Custom DaktGUI module with full geometry primitives and advanced widgets
8. [ ] **Export** - FBX/glTF export

### Phase 4: Overlay Features
9. [ ] **Overlay** - External overlay window
10. [ ] **OCR** - Text recognition

### Phase 5: Applications
11. [ ] **DaktExplorer** - File browser/exporter
12. [ ] **DaktOverlay** - Game overlay
13. [ ] **DaktParser** - Development/RE tool

---

## Completed: GUI Module ✅

### GUI Components
- [x] `Types.hpp` - Vec2, Rect, Color, Corner flags, Hexagon
- [x] `Context.hpp/.cpp` - GUI context with input state and window management
- [x] `DrawList.hpp/.cpp` - Immediate-mode draw commands, clipping, anti-aliasing
- [x] `Layout.hpp/.cpp` - Layout helpers, spacing, alignment
- [x] `Theme.hpp/.cpp` - Dark/light themes, color schemes
- [x] `Font.hpp/.cpp` - TrueType font loading and atlas building
- [x] `Input.hpp/.cpp` - Keyboard and mouse input processing
- [x] `Widgets.hpp/.cpp` - Buttons, checkboxes, sliders, text input, combos, trees
- [x] `Containers.hpp/.cpp` - Windows, panels, scrollbars, tabs, split panes
- [x] `DataWidgets.hpp/.cpp` - PropertyGrid, DataTable, HexView, Console, Timeline, NodeGraph
- [x] `D3D11Backend.hpp/.cpp` - DirectX 11 rendering backend

### GUI Features
- [x] Custom immediate-mode GUI (not ImGui)
- [x] All geometry primitives: Rect, Circle, Triangle, Hexagon, Arc, Bezier
- [x] Comprehensive widget library
- [x] Property editor with categories
- [x] Sortable/filterable data tables
- [x] Hex editor view
- [x] Console output widget
- [x] File browser widget
- [x] Animation timeline
- [x] Node-based graph editor
- [x] Theming with dark/light presets
- [x] Font rendering with anti-aliasing

---

## Completed: Parser Module ✅

### Parser Components
- [x] `Parser.hpp` - Main module header
- [x] `BinaryReader.hpp/.cpp` - Endian-aware reading, cursor, seek/skip/align
- [x] `PatternScanner.hpp/.cpp` - IDA-style pattern scanning with wildcards
- [x] `StructMapper.hpp/.cpp` - POD mapping, CryEngine structure definitions
- [x] `CryXmlParser.hpp/.cpp` - Binary XML V1/V2 format
- [x] `LocalizationParser.hpp/.cpp` - global.ini with UTF-8/16 support
- [x] `MaterialParser.hpp/.cpp` - .mtl material files (XML + CryXml)
- [x] `Crypto.hpp/.cpp` - AES-256-CBC (mbedtls) and Salsa20 encryption
- [x] `CryPakParser.hpp/.cpp` - P4K/PAK archives with encryption and zlib
- [x] `DataCoreParser.hpp/.cpp` - .dcb StarCitizen database files
- [x] `ObjectContainerParser.hpp/.cpp` - .socpak object container files
- [x] `ChunkFileParser.hpp/.cpp` - .cgf/.chr/.skin CryEngine geometry
- [x] `TextureParser.hpp/.cpp` - .dds texture files (DXT1-5, BC1-7)

### Parser Features
- [x] Endian-aware binary reading
- [x] Pattern scanning with wildcards
- [x] CryEngine binary XML parsing
- [x] Localization file parsing
- [x] Material file parsing
- [x] DataCore parsing with crypto
- [x] P4K archive parsing with zlib

---

## Completed: VFS Module ✅

### VFS Components
- [x] `VFS.hpp` - Main module header
- [x] `IFileProvider.hpp` - Abstract provider interface
- [x] `VirtualPath.hpp/.cpp` - Path normalization
- [x] `PhysicalProvider.hpp/.cpp` - Native filesystem mount
- [x] `MemoryProvider.hpp/.cpp` - In-memory files
- [x] `VirtualFileSystem.hpp/.cpp` - Mount points and dispatch
- [x] `FileWatcher.hpp/.cpp` - Change detection

### VFS Features
- [x] Unified file access across different sources
- [x] Mount point priorities for overlays
- [x] Case-insensitive path handling
- [x] Directory enumeration
- [x] Watch for file changes
- [ ] P4K archive provider (uses Parser/CryPakParser)
- [ ] PAK archive provider (uses Parser/CryPakParser)

---

## Next Up: GUI Module
| Export | tinygltf | ⬜ To fetch |
| Export | FBX SDK / OpenFBX | ⬜ To decide |
| OCR | Tesseract | ⬜ Optional |

---

## Testing Infrastructure

- [ ] Set up Google Test or Catch2
- [ ] Create test utilities module
- [ ] Add benchmark framework
- [ ] Set up CI/CD pipeline
- [ ] Add code coverage

---

## Documentation

- [x] ARCHITECTURE.md (top-level)
- [x] Core/ARCHITECTURE.md
- [x] Core/TODO.md
- [ ] README.md (project overview)
- [ ] BUILDING.md (build instructions)
- [ ] CONTRIBUTING.md (contribution guidelines)
- [ ] API reference (Doxygen)

---

## Star Citizen Specific

### File Formats to Support
- [ ] .p4k (game archives)
- [ ] .pak (CryPak archives)
- [ ] .socpak (Object containers)
- [ ] .dcb (DataCore binary)
- [ ] .cgf/.chr/.skin (CryEngine geometry)
- [ ] .mtl (Materials)
- [ ] .dds (Textures)
- [ ] global.ini (Localization)

### Overlay Features
- [ ] Screen text reading (OCR)
- [ ] Hotkey panels
- [ ] Info overlay
- [ ] Map overlay
- [ ] Inventory helper

---

## Notes

### Build Requirements
- C++23 compiler (MSVC 19.35+, GCC 13+, Clang 16+)
- CMake 3.20+
- Windows SDK (for Overlay module)

### Priority Guidelines
1. Get logging working early (essential for debugging)
2. VFS + Parser are critical path for tools
3. GUI needed before Export (need to preview)
4. Overlay/OCR can be parallel track

### Known Blockers
- FBX SDK licensing (consider OpenFBX alternative)
- Tesseract training for Star Citizen fonts
- EasyAntiCheat compatibility testing