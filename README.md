# DaktLib

A dependency-free, cross-platform, plug-and-play C++23 modular library suite for Star Citizen utilities. Modules are standalone, interface-driven, and EAC-safe (no injection, no process memory access, no graphics API hooks).

## Highlights
- Zero hard dependencies across all modules; opt-in integrations via DaktLib-Core interfaces
- Modern C++23 (concepts, std::expected-style Result, coroutines where useful)
- EAC-safe capture/overlay/OCR paths using only official OS APIs
- C API surfaces for every module to enable downstream bindings (e.g., C# via ClangSharp)
- Cross-platform targets: Windows, Linux, macOS (x64/ARM64 where applicable)
- FBX 7.5 exporter with USD-like stage/layer/prim model, no external SDKs

## Module Map
- Foundation: Core, Logger, Events
- Data & Storage: VFS, Parser, SQL, Export, Decrypt
- Services: Http, Capture
- Compute & Vision: ML, OCR
- Presentation: Overlay, GUI

Detailed architecture and diagrams live in [ARCHITECTURE.md](ARCHITECTURE.md). Per-module docs are in each submodule’s `README.md`.

## Current Status
See the master roadmap in [TODO.md](TODO.md). Snapshot:
- Scaffolded: Core, Logger, Events, VFS, Parser, SQL, Export, Decrypt, Http, Capture, ML, OCR, Overlay, GUI
- Near-term focus: FBX writer core, Logger/Events hardening, Http backend + TLS, one capture backend, Overlay surfaces + painter, Parser JSON/XML/P4K basics, SQL sqlite3 drop-in, ML/OCR Windows backend

## Project Layout
```
DaktLib/
├── ARCHITECTURE.md      # Architecture overview and diagrams
├── TODO.md              # Master roadmap and milestones
├── CMakeLists.txt       # Superbuild (adds module subdirectories)
├── CMakePresets.json    # Configure/build presets
├── DaktLib-*/           # Module subrepos (Core, Logger, Events, VFS, Parser, SQL, Export, Decrypt, Http, Capture, ML, OCR, Overlay, GUI)
└── README.md            # This file
```
Each module follows a consistent shape: `include/` public headers, `src/` implementations, optional `bindings/` (C/C#), and `tests/`.

## Getting Started
### Prerequisites
- CMake 4.2.1+
- Clang 16+ (or MSVC 19.36+, GCC 13+ as alternative targets)
- Ninja or your chosen generator

### Clone with Submodules
```
git clone https://github.com/Schuldakt/DaktLib.git
cd DaktLib
git submodule update --init --recursive
```

### Configure & Build (example)
```
# Configure a release build (adjust preset/toolchain as needed)
cmake --preset windows-release

# Build everything
cmake --build --preset windows-release
```
Common cache options (toggle in presets or -D):
- `DAKT_BUILD_ALL` (default ON)
- `DAKT_BUILD_CORE`, `DAKT_BUILD_LOGGER`, ... (per-module switches)

### Run Tests
```
cmake --build --preset windows-release --target test
ctest --preset windows-release
```

## Usage Sketch
Minimal pattern using Core + Logger (C++):
```cpp
#include "dakt/logger/Logger.hpp"
#include "dakt/core/Result.hpp"

int main() {
    dakt::logger::Logger logger;
    logger.info("Hello from DaktLib");

    auto r = dakt::core::Result<int>::ok(42);
    if (r.isOk()) {
        logger.info("Value: {}", r.value());
    }
    return 0;
}
```

C API handles follow the same pattern across modules (create/destroy, integer error codes, `dakt_get_last_error`).

## Design Notes
- Plug-and-play: bring your own logger/allocator/event bus by implementing Core interfaces
- No exceptions for routine flow; prefer `Result<T,E>` and `std::optional`
- Threading: single-threaded by default; thread-safe components are documented per module (e.g., Logger sinks, Events dispatch, SQL pools)

## Roadmap & Milestones
- v0.1.0: Core/Logger/Events usable
- v0.2.0: Data layer MVP (VFS, Parser, SQL, Decrypt)
- v0.3.0: Export + Http/Capture MVP
- v0.4.0: Vision/Compute (ML, OCR)
- v0.5.0: Presentation (Overlay, GUI basics)
- v1.0.0: Cross-platform polish, bindings, packaging

For full details and risks, read [TODO.md](TODO.md).

## Contributing
- Keep modules dependency-free; isolate platform code under `src/platform/`
- Add/update C API stubs alongside C++ APIs
- Run formatting (`clang-format`) and static checks (`clang-tidy`) before PRs
- Update module README/TODO plus [ARCHITECTURE.md](ARCHITECTURE.md) when altering scope or contracts
