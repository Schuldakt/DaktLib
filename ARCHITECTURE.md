# DaktLib Architecture

> A 100% dependency-free, cross-platform, plug-and-play C++23 modular library suite for Star Citizen utility development.

## Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              DaktLib Ecosystem                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    DaktLib-Core (Header-Only)                       │    │
│  │         ILogger, IAllocator, IEventBus, IRegionProvider             │    │
│  │              Result<T,E>, Span<T>, StringView                       │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│       ▲ (optional)    ▲ (optional)    ▲ (optional)    ▲ (optional)          │
│       │               │               │               │                     │
│  ┌────┴────┐    ┌─────┴────┐    ┌─────┴────┐    ┌─────┴────┐                │
│  │ Logger  │    │  Events  │    │  Export  │    │   VFS    │  Foundation    │
│  └─────────┘    └──────────┘    └──────────┘    └──────────┘                │
│                                                                             │
│  ┌─────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐                │
│  │ Parser  │    │   SQL    │    │   Http   │    │ Capture  │  Services      │
│  └─────────┘    └──────────┘    └──────────┘    └──────────┘                │
│                                       │               │                     │
│                                       ▼               ▼                     │
│  ┌─────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐                │
│  │   ML    │──▶ │   OCR    │◀───│ Overlay  │───▶│   GUI    │  Integration │
│  └─────────┘    └──────────┘    └──────────┘    └──────────┘                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Design Principles

### 1. Zero Hard Dependencies
Every module is standalone. Optional integrations use interfaces from DaktLib-Core.

### 2. Plug-and-Play Architecture
Users can:
- Use DaktLib implementations OR provide their own via interfaces
- Mix and match modules freely
- Replace any component (e.g., bring your own logger)

### 3. EAC Compliance (Anti-Cheat Safe)
Critical for Star Citizen utilities:
- **NO** DLL injection
- **NO** process memory reading/writing
- **NO** graphics API hooking
- **ONLY** official OS APIs for capture/overlay

### 4. Cross-Platform Support

| Platform | Compiler | Min Version |
|----------|----------|-------------|
| Windows x64 | MSVC | 19.36+ |
| Windows x64 | Clang | 16+ |
| Linux x64 | GCC | 13+ |
| Linux x64 | Clang | 16+ |
| macOS x64 | AppleClang | 15+ |
| macOS ARM64 | AppleClang | 15+ |

---

## Module Overview (aligned with current READMEs)

### Foundation
- **DaktLib-Core**: header-first interfaces (logging, allocation, events, regions) and lightweight types (`Result`, `Span`, `StringView`).
- **DaktLib-Logger**: multi-sink logging (console/file/async) with structured fields and C API.
- **DaktLib-Events**: type-safe event bus, priorities, cancellation; C API planned.

### Data & Storage
- **DaktLib-VFS**: virtual file system with mountable backends (physical, memory, mmap, zip/pak placeholders); streaming interfaces.
- **DaktLib-Parser**: zero-dependency parsing toolkit (JSON/XML/INI/CSV, binary streams, game/asset formats like P4K, CryXML, CryModel, MTL, DCB, SOC/SOCPAK).
- **DaktLib-SQL**: lightweight SQLite-style wrapper with connection pooling, query/schema builders, and C API (vendored amalgamation slot).
- **DaktLib-Export**: FBX 7.5 exporter with USD-like stage/layer/prim model, layered PBR materials, animation, optional ASCII debug output; no external SDK.
- **DaktLib-Decrypt**: CryEngine model chunk detection/decrypt/inspect (magic, chunk layouts, crypto/comp/hash), modular registries.

### Services
- **DaktLib-Http**: dependency-free HTTP client skeleton with platform-native backends (WinHTTP, CFNetwork, POSIX sockets), TLS abstraction, C API.
- **DaktLib-Capture**: EAC-safe screen/window capture scaffolding (DXGI, WGC, PipeWire, X11, ScreenCaptureKit placeholders).

### Compute & Vision
- **DaktLib-ML**: on-device inference helper with runtime-loaded providers (ONNX Runtime, DirectML, TensorRT), C API.
- **DaktLib-OCR**: OCR pipeline with pluggable backends (Windows.Media.Ocr, ONNX), preprocessing, EAC-safe (capture-only), C API.

### Presentation
- **DaktLib-Overlay**: OS-native translucent overlays (WS_EX_LAYERED/DirectComposition, X11/Wayland, NSWindow/CoreAnimation), no hooks/injection, C API.
- **DaktLib-GUI**: immediate-first GUI with optional retained tree, flex layout, SDF/MSDF text, and DX11/12/Vulkan/OpenGL/Metal backends; C API.

---

## Technology Stack

### Build System
- **CMake 4.2.1+** — Modern CMake with presets
- **Clang 16+** — Primary compiler
- **clang-format** — Code formatting
- **clang-tidy** — Static analysis

### C++ Standard
- **C++23** — Full feature set
  - `std::expected` for error handling
  - `std::format` for string formatting
  - Concepts for type constraints
  - Coroutines for async operations
  - `std::source_location` for logging

### Code Style
```cpp
// Namespaces: lowercase with scope
namespace dakt::module_name { }

// Types: PascalCase
class MyClass { };
struct MyStruct { };
enum class MyEnum { };

// Functions/Methods: camelCase
void doSomething();
auto getValue() -> int;

// Constants: SCREAMING_SNAKE_CASE
constexpr int MAX_BUFFER_SIZE = 4096;

// Member variables: m_ prefix
int m_value;

// Parameters/Locals: camelCase
void process(int inputValue) {
    int localVar = 0;
}
```

---

## Directory Structure

```
DaktLib/
├── .clang-format              # Code formatting rules
├── .clang-tidy                # Static analysis config
├── .gitmodules                # Submodule definitions
├── CMakeLists.txt             # Root superbuild
├── CMakePresets.json          # Build presets
├── ARCHITECTURE.md            # This file
├── TODO.md                    # Master roadmap
├── LICENSE                    # Project license
├── README.md                  # Project overview
│
├── DaktLib-Core/              # Header-only interfaces
├── DaktLib-Logger/            # Logging library
├── DaktLib-Events/            # Event bus
├── DaktLib-VFS/               # Virtual filesystem
├── DaktLib-Parser/            # Parsing toolkit (text + game/asset formats)
├── DaktLib-SQL/               # SQLite-style wrapper
├── DaktLib-Export/            # FBX exporter (USD-like model)
├── DaktLib-Decrypt/           # CryEngine chunk decrypt/inspect
├── DaktLib-Http/              # HTTP client
├── DaktLib-Capture/           # Screen/window capture
├── DaktLib-ML/                # Inference helper (runtime-loaded backends)
├── DaktLib-OCR/               # OCR engine (pluggable backends)
├── DaktLib-Overlay/           # Overlay windows (EAC-safe)
└── DaktLib-GUI/               # GUI framework (multi-backend)
```

### Per-Module Structure

```
DaktLib-XXX/
├── CMakeLists.txt             # Module build config
├── ARCHITECTURE.md            # Module architecture
├── TODO.md                    # Module roadmap
├── README.md                  # Module documentation
│
├── include/
│   └── dakt/
│       └── xxx/
│           ├── XXX.hpp        # Main include header
│           └── *.hpp          # Public headers
│
├── src/
│   ├── *.cpp                  # Implementation files
│   └── platform/
│       ├── windows/           # Windows-specific
│       ├── linux/             # Linux-specific
│       └── macos/             # macOS-specific
│
├── bindings/
│   ├── c/
│   │   └── daktxxx.h          # C API header
│   └── csharp/
│       └── DaktXXX/           # C# project
│
└── tests/
    ├── unit/                  # Unit tests
    └── integration/           # Integration tests
```

---

## C API & FFI Conventions

Every module exports a flat C API for language bindings:

### Handle Pattern
```c
// Opaque handle types
typedef struct DaktLogger* DaktLoggerHandle;

// Lifecycle
DaktLoggerHandle dakt_logger_create(void);
void dakt_logger_destroy(DaktLoggerHandle logger);

// Operations return error codes
int dakt_logger_log(DaktLoggerHandle logger, int level, const char* message);

// Error handling
const char* dakt_get_last_error(void);
```

### Export Macros
```c
#ifdef _WIN32
    #ifdef DAKT_XXX_EXPORTS
        #define DAKT_XXX_API __declspec(dllexport)
    #else
        #define DAKT_XXX_API __declspec(dllimport)
    #endif
#else
    #define DAKT_XXX_API __attribute__((visibility("default")))
#endif
```

### C# Binding Generation
- **ClangSharp** for auto-generation
- Split output by class/namespace
- Manual refinement layer for ergonomics

---

## Plug-and-Play Logger Pattern

All modules accept an optional logger without hard dependency:

### Option 1: Template Policy (Zero Overhead)
```cpp
template<typename LoggerPolicy = NullLogger>
class SomeModule {
    LoggerPolicy m_logger;
public:
    void doWork() {
        m_logger.info("Doing work...");
    }
};
```

### Option 2: Interface Injection (Runtime Flexibility)
```cpp
class SomeModule {
    dakt::core::ILogger* m_logger = nullptr;
public:
    void setLogger(dakt::core::ILogger* logger) {
        m_logger = logger;
    }
    void doWork() {
        if (m_logger) m_logger->info("Doing work...");
    }
};
```

---

## Error Handling

### Result Type (from DaktLib-Core)
```cpp
template<typename T, typename E = std::string>
class Result {
public:
    static Result ok(T value);
    static Result err(E error);
    
    bool isOk() const;
    bool isErr() const;
    
    T& value();
    E& error();
    
    // Monadic operations
    template<typename F> auto map(F&& f) -> Result<...>;
    template<typename F> auto andThen(F&& f) -> ...;
    T valueOr(T defaultVal) const;
};
```

### No Exceptions Policy
- Use `Result<T, E>` for recoverable errors
- Use `std::optional<T>` for absence of value
- Reserve exceptions for truly exceptional cases (OOM, corruption)

---

## Threading Model

### Default: Single-Threaded
Most modules are single-threaded by default for simplicity.

### Thread-Safe Components
Explicitly documented when thread-safe:
- `dakt::logger::Logger` — Thread-safe with internal synchronization
- `dakt::events::EventBus` — Thread-safe dispatch
- `dakt::sql::ConnectionPool` — Thread-safe checkout/return

### Async Operations
Use C++20 coroutines where applicable:
```cpp
dakt::http::Task<Response> fetchData() {
    auto response = co_await client.get("https://api.example.com/data");
    co_return response;
}
```

---

## Build Configuration

### CMake Superbuild
```cmake
cmake_minimum_required(VERSION 4.2.1)
project(DaktLib VERSION 1.0.0)

option(DAKT_BUILD_ALL "Build all modules" ON)
option(DAKT_BUILD_CORE "Build DaktLib-Core" ON)
option(DAKT_BUILD_LOGGER "Build DaktLib-Logger" ON)
# ... etc

if(DAKT_BUILD_CORE OR DAKT_BUILD_ALL)
    add_subdirectory(DaktLib-Core)
endif()
# ... etc
```

### CMake Presets
```json
{
    "version": 6,
    "configurePresets": [
        {
            "name": "windows-release",
            "generator": "Ninja",
            "binaryDir": "${sourceDir}/build/windows-release",
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "Release",
                "CMAKE_CXX_COMPILER": "clang++"
            }
        }
    ]
}
```

---

## Testing Strategy

### Unit Tests
- Framework: Catch2 or doctest (header-only)
- Coverage target: 80%+
- Run in CI on every PR

### Integration Tests
- Test module interactions
- Platform-specific tests

### Visual Regression (GUI)
- Screenshot comparison
- Automated with CI

---

## CI/CD Pipeline

### GitHub Actions Matrix
```yaml
strategy:
  matrix:
    os: [windows-latest, ubuntu-latest, macos-latest]
    compiler: [msvc, clang, gcc]
    exclude:
      - os: windows-latest
        compiler: gcc
      - os: macos-latest
        compiler: msvc
```

### Pipeline Stages
1. **Build** — Compile all configurations
2. **Test** — Run unit and integration tests
3. **Lint** — clang-tidy static analysis
4. **Package** — Generate NuGet/CMake packages

---

## Versioning

### Semantic Versioning
- **MAJOR** — Breaking API changes
- **MINOR** — New features, backward compatible
- **PATCH** — Bug fixes

### Per-Module Versioning
Each module has independent version numbers.

### Compatibility Matrix
Documented which module versions work together.

---

## License Considerations

### DaktLib Modules
- Intended: Permissive license (MIT or Apache 2.0)

### Third-Party Dependencies
| Dependency | License | Module |
|------------|---------|--------|
| SQLite | Public Domain | DaktLib-SQL |
| ONNX Runtime | MIT | DaktLib-ML |

All dependencies chosen for license compatibility.
