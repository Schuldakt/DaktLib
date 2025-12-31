# DaktLib TODO

## Module Dependency Graph (Minimal Dependencies)

Each module is designed to be independently usable in any C++ project:

```
Core (zero external deps)
├── Logger (Core only)
├── Events (Core only)
├── Config (Core only)
├── VFS (Core only)
├── Parser (Core only + mbedtls/zlib for crypto/compression)
├── GUI (Core only)
├── Export (Core only)
├── Overlay (Core + Logger - Logger is optional private dep)
└── OCR (Core + Logger - Logger is optional private dep)
```

**Note:** Logger in Overlay and OCR is a PRIVATE dependency - users of those modules
don't need Logger unless they want logging.

---

## Module Implementation Status

| Module | Status | Headers | Implementation | Tests | Docs |
|--------|--------|---------|----------------|-------|------|
| Core | ✅ Complete | ✅ | ✅ | ⬜ | ✅ |
| Logger | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| Events | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| Config | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| VFS | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| Parser | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| GUI | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| Export | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| Overlay | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |
| OCR | ✅ Complete | ✅ | ✅ | ⬜ | ⬜ |

**Legend:** ✅ Complete | 🔨 In Progress | ⬜ Not Started

---

## Implementation Order

### Phase 1: Foundation ✅ COMPLETE
1. [x] **Core** - Types, Memory, String, Buffer, Hash, Time, FileSystem, Platform
2. [x] **Logger** - Multi-sink logging with levels and formatting
3. [x] **Events** - EventBus, Signal/Slot, queued events
4. [x] **Config** - INI/TOML/JSON parsing (built-in), hot-reload via ConfigWatcher

### Phase 2: Data Access ✅ COMPLETE
5. [x] **VFS** - Virtual file system with mount points, P4K/PAK archive providers, overlay support
6. [x] **Parser** - Star Citizen file format handlers (core parsers complete, crypto pending)

### Phase 3: UI & Tools ✅ COMPLETE
7. [x] **GUI** - Custom DaktGUI module with full geometry primitives and advanced widgets
8. [x] **Export** - glTF/GLB and OBJ export with texture conversion and mesh optimization

### Phase 4: Overlay Features ✅ COMPLETE
9. [x] **Overlay** - External overlay window (transparent layered window, hotkeys, screen capture)
10. [x] **OCR** - Text recognition (Windows.Media.Ocr, preprocessing, region caching)

### Phase 5: Applications
11. [x] **DaktExplorer** - File browser/exporter (complete with full GUI support)
12. [ ] **DaktOverlay** - Game overlay
13. [ ] **DaktParser** - Development/RE tool

---

## Completed: OCR Module ✅

### OCR Components
- [x] `OCR.hpp` - Main module header
- [x] `OcrResult.hpp` - Recognition result structures (OcrWord, OcrLine, OcrResult)
- [x] `IOcrEngine.hpp` - Abstract engine interface with OcrImage, OcrEngineConfig
- [x] `WindowsOcrEngine.hpp/.cpp` - Windows.Media.Ocr WinRT implementation
- [x] `ImagePreprocessor.hpp/.cpp` - Full preprocessing pipeline (~700 lines)
- [x] `OcrRegion.hpp/.cpp` - Region definition and caching system

### OCR Features
- [x] Windows.Media.Ocr backend (Windows 10+)
- [x] Region-based OCR (recognizeRegion method)
- [x] Confidence scores (word/line/result level)
- [x] Image preprocessing pipeline:
  - [x] Grayscale conversion (ITU-R BT.601)
  - [x] Bilinear scaling
  - [x] Contrast enhancement
  - [x] Binarization (Fixed, Otsu, Adaptive)
  - [x] Denoising (Median, Gaussian, Bilateral)
  - [x] Morphological operations (dilate/erode)
- [x] Result caching with TTL
- [x] Predefined Star Citizen UI regions (SCRegions namespace)

---

## Completed: Overlay Module ✅

### Overlay Components
- [x] `OverlayWindow.hpp/.cpp` - Transparent layered window with D3D11 rendering
- [x] `HotkeyManager.hpp/.cpp` - Global hotkey registration with RegisterHotKey
- [x] `ProcessDetector.hpp/.cpp` - Game process detection using Toolhelp32 and EnumWindows
- [x] `WindowTracker.hpp/.cpp` - Target window position/size tracking
- [x] `ScreenCapture.hpp/.cpp` - BitBlt and DXGI Desktop Duplication capture
- [x] `Panel.hpp/.cpp` - Panel interface and base implementations

### Overlay Features
- [x] Transparent overlay window (WS_EX_LAYERED | WS_EX_TOPMOST | WS_EX_TRANSPARENT)
- [x] D3D11 with premultiplied alpha swap chain
- [x] DwmExtendFrameIntoClientArea for transparency
- [x] Global hotkeys with callback system
- [x] Process detection and monitoring
- [x] Window position/state tracking
- [x] Multi-monitor support
- [x] BitBlt GDI capture
- [x] DXGI Desktop Duplication capture
- [x] Image format conversion (BGRA8, RGBA8, Grayscale)
- [x] Bilinear scaling and cropping utilities
- [x] Flexible panel system with anchoring

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
- [X] P4K archive provider (uses Parser/CryPakParser)
- [X] PAK archive provider (uses Parser/CryPakParser)

---

## Completed: Export Module ✅

### Export Components
- [x] `Types.hpp` - TextureFormat, TextureInfo, Material, Mesh, Primitive types
- [x] `Scene.hpp/.cpp` - Scene graph with Vec2/3/4, Quat, Mat4, AABB math types
- [x] `SceneBuilder.hpp/.cpp` - Scene construction and validation
- [x] `IExporter.hpp` - Base exporter interface with progress callbacks
- [x] `GltfExporter.hpp/.cpp` - glTF 2.0 and GLB binary export
- [x] `ObjExporter.hpp/.cpp` - Wavefront OBJ/MTL export
- [x] `TextureConverter.hpp/.cpp` - DDS decompression and format conversion
- [x] `MeshOptimizer.hpp/.cpp` - Mesh optimization and LOD generation

### Export Features
- [x] Complete scene graph (nodes, meshes, materials, skins, animations)
- [x] glTF 2.0 JSON export
- [x] GLB binary format export
- [x] OBJ geometry with MTL materials
- [x] PBR material model
- [x] Coordinate system conversion (Z-up to Y-up)
- [x] BC1-BC5 texture decompression
- [x] Mipmap generation (box filter)
- [x] Bilinear/nearest texture resize
- [x] Vertex welding (hash-based duplicate removal)
- [x] Vertex cache optimization (Tom Forsyth algorithm)
- [x] LOD generation (quadric error metrics)

---

## Next Up: Overlay Module
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