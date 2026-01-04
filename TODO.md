# DaktLib Master TODO

> Master roadmap for the DaktLib modular library suite.

## Legend

| Symbol | Meaning |
|--------|---------|
| ⬜ | Not started |
| 🔄 | In progress |
| ✅ | Complete |
| ❌ | Blocked/Cancelled |
| 🔴 | Critical priority |
| 🟡 | Medium priority |
| 🟢 | Low priority |

---

## Phase 1: Foundation (Weeks 1-6)

> Core infrastructure that all other modules depend on.

### DaktLib-Core 🔴
- [ ] Project structure and CMakeLists.txt
- [ ] `Result<T, E>` implementation with monadic operations
- [ ] `Span<T>` non-owning view
- [ ] `StringView` implementation
- [ ] `ILogger` interface
- [ ] `IAllocator` interface
- [ ] `IEventBus` interface
- [ ] `ISerializable` interface
- [ ] `IRegionProvider` interface
- [ ] C++23 concepts (`Loggable`, `Serializable`)
- [ ] Unit tests
- [ ] Documentation

### DaktLib-Logger 🔴
- [ ] Project structure and CMakeLists.txt
- [ ] `LogLevel` enum and `LogRecord` struct
- [ ] `ISink` interface
- [ ] `ConsoleSink` with ANSI colors
- [ ] `FileSink` with rotation
- [ ] `AsyncSink` wrapper
- [ ] `Logger` class implementing `ILogger`
- [ ] Structured logging support
- [ ] `std::source_location` integration
- [ ] C API (`dakt_logger_*`)
- [ ] Unit tests
- [ ] Documentation

### DaktLib-Events 🔴
- [ ] Project structure and CMakeLists.txt
- [ ] Type-safe event registration
- [ ] `EventBus` implementing `IEventBus`
- [ ] Sync dispatch
- [ ] Async dispatch
- [ ] Event priorities
- [ ] Event cancellation
- [ ] RAII `Subscription` class
- [ ] Weak reference support
- [ ] C API (`dakt_events_*`)
- [ ] Unit tests
- [ ] Documentation

### Infrastructure 🔴
- [ ] Root CMakeLists.txt superbuild
- [ ] CMakePresets.json
- [ ] .clang-format configuration
- [ ] .clang-tidy configuration
- [ ] GitHub Actions CI/CD pipeline
- [ ] Root README.md

---

## Phase 2: Data Layer (Weeks 7-14)

> File handling, parsing, database, and export capabilities.

### DaktLib-VFS 🟡
- [ ] Project structure and CMakeLists.txt
- [ ] `IStream` interface
- [ ] `IBackend` interface
- [ ] `Path` cross-platform normalization
- [ ] Physical filesystem backend
- [ ] Memory backend
- [ ] Memory-mapped backend
- [ ] ZIP archive backend
- [ ] VFS mount system with priorities
- [ ] File watching (Windows/Linux/macOS)
- [ ] Async I/O support
- [ ] C API (`dakt_vfs_*`)
- [ ] Unit tests
- [ ] Documentation

### DaktLib-Parser 🟡
- [ ] Project structure and CMakeLists.txt
- [ ] Zero-copy `Reader` class
- [ ] SAX handler interface
- [ ] DOM node types
- [ ] JSON parser (RFC 8259)
- [ ] JSON writer
- [ ] XML parser
- [ ] XML writer
- [ ] INI parser
- [ ] CSV parser
- [ ] Binary reader/writer (endian-aware)
- [ ] Error recovery modes
- [ ] C API (`dakt_parser_*`)
- [ ] Unit tests
- [ ] Documentation

### DaktLib-SQL 🟡
- [ ] Project structure and CMakeLists.txt
- [ ] Vendor SQLite (public domain)
- [ ] `Connection` RAII wrapper
- [ ] `PreparedStatement` with binding
- [ ] `ResultSet` iteration
- [ ] `ConnectionPool` thread-safe
- [ ] `Transaction` RAII
- [ ] Query builder (fluent API)
- [ ] Schema migrations
- [ ] Async query execution
- [ ] C API (`dakt_sql_*`)
- [ ] Unit tests
- [ ] Documentation

### DaktLib-Export 🟡
- [ ] Project structure and CMakeLists.txt
- [ ] `IExporter` interface
- [ ] CSV exporter
- [ ] JSON exporter
- [ ] XML exporter
- [ ] HTML table exporter
- [ ] Markdown table exporter
- [ ] `StreamWriter` for large datasets
- [ ] `ColumnMapper` transformations
- [ ] `BatchExporter`
- [ ] C API (`dakt_export_*`)
- [ ] Unit tests
- [ ] Documentation

---

## Phase 3: Services (Weeks 15-26)

> HTTP client and screen capture.

### DaktLib-Http 🟡
- [ ] Project structure and CMakeLists.txt
- [ ] URL parser
- [ ] `Request` / `Response` types
- [ ] `HttpClient` class
- [ ] HTTP/1.1 protocol implementation
- [ ] Windows backend (WinHTTP)
- [ ] Linux backend (POSIX sockets)
- [ ] macOS backend (CFNetwork)
- [ ] TLS support (platform-native)
- [ ] Connection pooling
- [ ] Timeouts and retry policies
- [ ] C++20 coroutine support
- [ ] Progress callbacks
- [ ] Cookie handling
- [ ] HTTP/2 support (stretch goal)
- [ ] C API (`dakt_http_*`)
- [ ] Unit tests
- [ ] Documentation

### DaktLib-Capture 🔴
- [ ] Project structure and CMakeLists.txt
- [ ] `ICaptureBackend` interface
- [ ] `CaptureSession` class
- [ ] `Frame` data structure
- [ ] Windows: DXGI Desktop Duplication
- [ ] Windows: Windows.Graphics.Capture
- [ ] Linux: PipeWire backend
- [ ] Linux: X11 XShm/XComposite
- [ ] macOS: ScreenCaptureKit
- [ ] macOS: CGWindowListCreateImage fallback
- [ ] Multi-monitor support
- [ ] Cursor capture option
- [ ] Frame rate control
- [ ] Pixel format conversion
- [ ] C API (`dakt_capture_*`)
- [ ] EAC compliance verification
- [ ] Unit tests
- [ ] Documentation

---

## Phase 4: AI/Vision (Weeks 27-34)

> Machine learning and OCR capabilities.

### DaktLib-ML 🟡
- [ ] Project structure and CMakeLists.txt
- [ ] `Tensor` class with shape/dtype
- [ ] `IInferenceEngine` interface
- [ ] ONNX Runtime backend
- [ ] DirectML provider (Windows)
- [ ] TensorRT backend (optional)
- [ ] CPU fallback
- [ ] Model loading/caching
- [ ] Memory-mapped models
- [ ] Session pooling
- [ ] Batch inference
- [ ] Preprocessing utilities
- [ ] C API (`dakt_ml_*`)
- [ ] Unit tests
- [ ] Documentation

### DaktLib-OCR 🔴
- [ ] Project structure and CMakeLists.txt
- [ ] `IOcrEngine` interface
- [ ] `OcrResult` / `OcrWord` types
- [ ] Windows.Media.Ocr backend
- [ ] ONNX ML backend (via DaktLib-ML)
- [ ] Image preprocessor pipeline
  - [ ] Grayscale conversion
  - [ ] Scaling
  - [ ] Binarization (Otsu, adaptive)
  - [ ] Denoising
  - [ ] Morphological operations
- [ ] `IRegionProvider` integration
- [ ] `RegionCache` with change detection
- [ ] DaktLib-Capture integration
- [ ] C API (`dakt_ocr_*`)
- [ ] Unit tests
- [ ] Documentation

---

## Phase 5: Presentation (Weeks 35-52+)

> Overlay and GUI systems.

### DaktLib-Overlay 🟡
- [ ] Project structure and CMakeLists.txt
- [ ] `IOverlayBackend` interface
- [ ] `OverlayWindow` class
- [ ] Windows: Layered windows (WS_EX_LAYERED)
- [ ] Windows: DWM composition
- [ ] Linux: X11 composite
- [ ] Linux: Wayland layer-shell
- [ ] macOS: NSWindow transparent
- [ ] Click-through support
- [ ] Multi-monitor support
- [ ] DPI awareness
- [ ] DaktLib-Capture integration
- [ ] DaktLib-GUI integration
- [ ] C API (`dakt_overlay_*`)
- [ ] EAC compliance verification
- [ ] Unit tests
- [ ] Documentation

### DaktLib-GUI 🟡
- [ ] Project structure and CMakeLists.txt
- [ ] **Core**
  - [ ] `Context` state management
  - [ ] `IDStack` widget identification
  - [ ] `InputState` handling
  - [ ] `DrawList` command buffer
- [ ] **Layout Engine**
  - [ ] Flexbox container
  - [ ] Justify/align content
  - [ ] Grow/shrink/basis
  - [ ] Wrapping and nesting
  - [ ] Padding/margin/gap
- [ ] **Primitives**
  - [ ] Rectangle (with border radius)
  - [ ] Circle, Ellipse
  - [ ] Hexagon, Polygon
  - [ ] Lines, Polylines
  - [ ] Bezier curves
  - [ ] Anti-aliasing
- [ ] **Font System**
  - [ ] TrueType parsing
  - [ ] SDF generation
  - [ ] Glyph atlas
  - [ ] Text rendering
  - [ ] Kerning
- [ ] **Widgets**
  - [ ] Button, ImageButton
  - [ ] Label, Text
  - [ ] TextInput (single/multi-line)
  - [ ] Checkbox, Radio
  - [ ] Slider, RangeSlider
  - [ ] ProgressBar
  - [ ] Dropdown, ComboBox
  - [ ] ListBox, TreeView
  - [ ] Tabs
  - [ ] Tooltip, Modal
  - [ ] Context menu
  - [ ] Scrollable container
- [ ] **Animation**
  - [ ] Property animator
  - [ ] Easing functions
  - [ ] Keyframe animations
  - [ ] Transitions
  - [ ] Chaining
- [ ] **Styling**
  - [ ] Theme system
  - [ ] Dark/Light themes
  - [ ] Style overrides
- [ ] **Render Backends**
  - [ ] OpenGL 4.5+
  - [ ] DirectX 11
  - [ ] DirectX 12
  - [ ] Vulkan
  - [ ] Metal
- [ ] **Advanced**
  - [ ] Custom shaders
  - [ ] Texture atlasing
  - [ ] Retained mode containers
- [ ] C API (`dakt_gui_*`)
- [ ] Unit tests
- [ ] Documentation

---

## Phase 6: Bindings & Packaging (Ongoing)

> C# bindings and distribution.

### C# Bindings 🟢
- [ ] ClangSharp integration
- [ ] Per-module binding generation
- [ ] Manual refinement layer
- [ ] NuGet package structure
- [ ] DaktLib.Core.CSharp
- [ ] DaktLib.Logger.CSharp
- [ ] DaktLib.Events.CSharp
- [ ] DaktLib.VFS.CSharp
- [ ] DaktLib.Parser.CSharp
- [ ] DaktLib.SQL.CSharp
- [ ] DaktLib.Export.CSharp
- [ ] DaktLib.Http.CSharp
- [ ] DaktLib.Capture.CSharp
- [ ] DaktLib.ML.CSharp
- [ ] DaktLib.OCR.CSharp
- [ ] DaktLib.Overlay.CSharp
- [ ] DaktLib.GUI.CSharp
- [ ] Integration tests
- [ ] Documentation

### Packaging 🟢
- [ ] CMake install targets
- [ ] CMake find_package support
- [ ] vcpkg port
- [ ] Conan recipe
- [ ] NuGet packages (native)
- [ ] GitHub Releases automation

---

## Milestones

| Milestone | Target | Modules |
|-----------|--------|---------|
| **v0.1.0** | Week 6 | Core, Logger, Events |
| **v0.2.0** | Week 14 | + VFS, Parser, SQL, Export |
| **v0.3.0** | Week 26 | + Http, Capture |
| **v0.4.0** | Week 34 | + ML, OCR |
| **v0.5.0** | Week 44 | + Overlay |
| **v1.0.0** | Week 52+ | + GUI, full C# bindings |

---

## Estimated Effort by Module

| Module | Complexity | Weeks |
|--------|------------|-------|
| DaktLib-Core | Low | 1-2 |
| DaktLib-Logger | Medium | 2-3 |
| DaktLib-Events | Medium | 2-3 |
| DaktLib-VFS | High | 8-12 |
| DaktLib-Parser | High | 6-8 |
| DaktLib-SQL | Medium | 4-5 |
| DaktLib-Export | Medium | 3-4 |
| DaktLib-Http | High | 16-18 |
| DaktLib-Capture | High | 8-12 |
| DaktLib-ML | High | 6-8 |
| DaktLib-OCR | High | 6-8 |
| DaktLib-Overlay | Medium | 6-8 |
| DaktLib-GUI | Very High | 27-40 |

**Total: ~95-130 weeks** (parallel work can reduce this significantly)

---

## Risk Register

| Risk | Impact | Mitigation |
|------|--------|------------|
| EAC false positive | High | Test with live game, document compliance |
| Platform API changes | Medium | Abstract behind interfaces |
| C# binding complexity | Medium | Start with ClangSharp, refine manually |
| GUI scope creep | High | Strict MVP definition per phase |
| Performance issues | Medium | Profile early, benchmark continuously |

---

## Notes

- Start with Windows as primary platform (Star Citizen focus)
- Linux/macOS support can follow
- GUI module is largest effort — consider splitting phases
- C# bindings should be generated incrementally per module
