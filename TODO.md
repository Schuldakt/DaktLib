# DaktLib TODO

## Module Implementation Status

| Module | Status | Headers | Implementation | Tests | Docs |
|--------|--------|---------|----------------|-------|------|
| Core | ✅ Complete | ✅ | ✅ | ⬜ | ✅ |
| Logger | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| Events | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| Config | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| VFS | ✅ Core Complete | ✅ | ✅ | ⬜ | ⬜ |
| Parser | ✅ Core Complete | ✅ | ✅ | ⬜ | ⬜ |
| GUI | ⬜ Not Started | ⬜ | ⬜ | ⬜ | ⬜ |
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

### Phase 3: UI & Tools
7. [ ] **GUI** - Custom DaktGUI module integration (needs Rectangles, Squares, Circles, Hexagons, Triangles. Also needs panels, grids, tables, scrollbars, windows, etc.) Must be better and have more features than ImGUI.
8. [ ] **Export** - FBX/glTF export

### Phase 4: Overlay Features
9. [ ] **Overlay** - External overlay window
10. [ ] **OCR** - Text recognition

### Phase 5: Applications
11. [ ] **DaktExplorer** - File browser/exporter
12. [ ] **DaktOverlay** - Game overlay
13. [ ] **DaktParser** - Development/RE tool

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

### Parser Features
- [x] Endian-aware binary reading
- [x] Pattern scanning with wildcards
- [x] CryEngine binary XML parsing
- [x] Localization file parsing
- [x] Material file parsing
- [ ] DataCore parsing (pending crypto)
- [ ] P4K archive parsing (pending zlib)

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
- [ ] P4K archive provider (pending zlib)
- [ ] PAK archive provider (pending zlib)

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