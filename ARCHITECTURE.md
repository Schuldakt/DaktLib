# DaktLib Architecture

## Overview

DaktLib is a modular C++23 library ecosystem designed for Star Citizen modding tools. The library provides a complete stack from low-level utilities to high-level game file parsing, GUI rendering, and overlay systems.

## Design Philosophy

### Core Principles

1. **Modern C++23** - Leverages `std::expected`, `std::optional`, `std::span`, `std::format`, concepts, ranges, and other C++23 features for safety and expressiveness.

2. **Zero-Cost Abstractions** - No runtime overhead for unused features. Templates and concepts over virtual dispatch where appropriate.

3. **Modular Design** - Each module is an independent static library with explicit dependencies. Enable only what you need.

4. **No Exceptions** - Error handling via `Result<T, E>` (alias for `std::expected`). Assertions for programming errors, results for runtime failures.

5. **Explicit Over Implicit** - Dependencies injected, not discovered. No hidden singletons or global state (except where explicitly documented).

6. **Windows-First, Cross-Platform Ready** - Primary target is Windows (Star Citizen platform), but Linux/macOS support maintained where possible.

## Module Hierarchy

```
┌─────────────────────────────────────────────────────────────────────┐
│                          APPLICATIONS                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │ DaktOverlay  │  │ DaktExplorer │  │  DaktParser  │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
└─────────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────────┐
│                         HIGH-LEVEL MODULES                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │    Export    │  │   Overlay    │  │     OCR      │               │
│  │  (FBX/glTF)  │  │  (Windows)   │  │  (Tesseract) │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
└─────────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────────┐
│                         MID-LEVEL MODULES                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │     GUI      │  │    Parser    │  │     VFS      │               │
│  │   (ImGui)    │  │  (SC Files)  │  │  (Virtual)   │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
└─────────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────────┐
│                         FOUNDATION MODULES                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │    Logger    │  │    Events    │  │    Config    │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
└─────────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────────┐
│                              CORE                                   │
│  Types • Memory • String • Buffer • Hash • Time • FileSystem        │
│  Platform • Macros                                                  │
└─────────────────────────────────────────────────────────────────────┘
```

## Module Descriptions

### Core (No Dependencies)
Foundation types and utilities used by all other modules:
- **Types.hpp** - `Result<T,E>`, `Option<T>`, `Span<T>`, integer aliases, concepts
- **Memory.hpp** - Allocators (Heap, Arena, Pool), smart pointers, alignment utilities
- **String.hpp** - String manipulation, parsing, encoding (UTF-8, Base64, Hex)
- **Buffer.hpp** - Binary buffer, reader/writer with endianness support
- **Hash.hpp** - FNV-1a, CRC32, XXHash, MurmurHash, compile-time hashing
- **Time.hpp** - High-resolution timing, timestamps, stopwatch, frame timer
- **FileSystem.hpp** - File I/O, path utilities, directory traversal, memory mapping
- **Platform.hpp** - OS detection, system info, environment, byte swapping

### Logger (→ Core)
Multi-sink logging system:
- Log levels (Trace, Debug, Info, Warn, Error, Fatal)
- Sinks: Console, File, RingBuffer, Callback
- Formatting with `std::format`
- Thread-safe, lock-free ring buffer for async logging

### Events (→ Core)
Event system for decoupled communication:
- Type-safe event bus
- Signal/slot connections
- Queued events for cross-thread communication
- Scoped subscriptions (auto-unsubscribe)

### Config (→ Core)
Configuration management:
- TOML, JSON, INI format support
- Type-safe value access
- Hot-reload with file watching
- Hierarchical configuration with overrides

### VFS (→ Core, Config, Logger)
Virtual file system:
- Mount points and providers
- Providers: Physical, Memory, Pak, P4k, Overlay
- Unified path resolution
- File watching for hot-reload

### Parser (→ Core, Logger, VFS)
Star Citizen file format parsing:
- Binary reader with pattern scanning
- Crypto: AES, Salsa20 decryption
- Format handlers:
  - DataCore (game database)
  - CryPak (.pak archives)
  - ObjectContainer (.socpak)
  - ChunkFile (.cgf, .chr, .skin)
  - Material, Texture, Localization
- Discovery tools for reverse engineering

### GUI (→ Core, Logger, Events)
Immediate-mode GUI system:
- Dear ImGui integration
- Backends: D3D11, D3D12, OpenGL, Vulkan
- Custom widgets:
  - PropertyGrid, TreeList, DataTable
  - HexView, ImageView, Timeline
  - NodeEditor, CodeEditor
- Theming and font management

### Overlay (→ Core, Logger, GUI)
External overlay system (Windows-only):
- Transparent, click-through window
- Global hotkey management
- Process/window detection and tracking
- Screen capture (BitBlt, DXGI, WGC)

### OCR (→ Core, Logger)
Text recognition:
- Tesseract and Windows.Media.Ocr engines
- Image preprocessing
- Game-specific font training
- UI region detection

### Export (→ Core, Logger, Parser)
3D asset export:
- Scene graph construction
- Exporters: FBX, glTF, OBJ, COLLADA
- Texture conversion
- Mesh optimization

## Build System

CMake-based with per-module options:

```cmake
option(DAKT_BUILD_CORE      "Build Core module"      ON)
option(DAKT_BUILD_LOGGER    "Build Logger module"    ON)
option(DAKT_BUILD_EVENTS    "Build Events module"    ON)
option(DAKT_BUILD_CONFIG    "Build Config module"    ON)
option(DAKT_BUILD_VFS       "Build VFS module"       ON)
option(DAKT_BUILD_PARSER    "Build Parser module"    ON)
option(DAKT_BUILD_GUI       "Build GUI module"       ON)
option(DAKT_BUILD_OVERLAY   "Build Overlay module"   ON)
option(DAKT_BUILD_OCR       "Build OCR module"       ON)
option(DAKT_BUILD_EXPORT    "Build Export module"    ON)
option(DAKT_BUILD_TESTS     "Build unit tests"       ON)
```

Each module produces a static library with namespaced target:
- `Dakt::Core`, `Dakt::Logger`, `Dakt::GUI`, etc.

## External Dependencies

| Module  | Dependencies                           |
|---------|----------------------------------------|
| Core    | None (C++23 standard library only)     |
| Config  | toml++ (header-only), nlohmann/json    |
| Logger  | None (optional: fmt for older compilers)|
| VFS     | zlib (pak decompression)               |
| Parser  | OpenSSL or mbedtls (crypto)            |
| GUI     | Dear ImGui, GLFW or SDL2               |
| Overlay | Windows SDK only                       |
| OCR     | Tesseract or Windows SDK               |
| Export  | FBX SDK or OpenFBX, tinygltf           |

## Error Handling Strategy

```cpp
// Use Result<T, E> for operations that can fail
Result<Buffer, Error> readFile(StringView path);

// Use Option<T> for values that may not exist
Option<String> getEnv(StringView name);

// Use assertions for programming errors
DAKT_ASSERT(ptr != nullptr);
DAKT_ASSERT_MSG(index < size, "Index out of bounds");

// Pattern: Early return with error propagation
auto loadConfig(StringView path) -> Result<Config, Error> {
    auto data = readFile(path);
    if (!data) return std::unexpected(data.error());
    
    auto parsed = parseToml(*data);
    if (!parsed) return std::unexpected(parsed.error());
    
    return *parsed;
}
```

## Naming Conventions

- **Types**: PascalCase (`BufferReader`, `FileInfo`)
- **Functions**: camelCase (`readFile`, `getSystemInfo`)
- **Constants**: PascalCase or SCREAMING_SNAKE (`MaxPathLength`, `DAKT_VERSION`)
- **Namespaces**: lowercase (`dakt::core`, `dakt::core::string`)
- **Files**: PascalCase matching primary type (`Buffer.hpp`, `FileSystem.cpp`)
- **Macros**: DAKT_ prefix, SCREAMING_SNAKE (`DAKT_ASSERT`, `DAKT_PLATFORM_WINDOWS`)

## Thread Safety

- **Core types**: Thread-safe for independent instances, not for shared access
- **Logger**: Thread-safe (lock-free ring buffer for async)
- **Events**: Main-thread only unless explicitly documented
- **GUI**: Main-thread only (ImGui limitation)
- **File operations**: Thread-safe for independent files

## Memory Management

1. **Stack allocation** preferred for small, fixed-size data
2. **Arena allocator** for temporary allocations with bulk free
3. **Pool allocator** for fixed-size objects with frequent alloc/free
4. **Heap allocator** as fallback, with optional tracking in debug builds

## Future Considerations

- Coroutine support for async file I/O
- GPU compute for texture processing
- Plugin system for format handlers
- Network layer for multiplayer tool features