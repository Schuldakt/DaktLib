# DaktLib Architecture

> A modular, dependency-free library ecosystem for Star Citizen utility development

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Rust](https://img.shields.io/badge/rust-1.75+-orange.svg)](https://www.rust-lang.org/)

---

## Table of Contents

1. [Overview](#overview)
2. [Design Principles](#design-principles)
3. [Language & Tooling](#language--tooling)
4. [Workspace Structure](#workspace-structure)
5. [Module Dependency Graph](#module-dependency-graph)
6. [Plug-and-Play Architecture](#plug-and-play-architecture)
7. [Cross-Cutting Concerns](#cross-cutting-concerns)
8. [C# Bindings Strategy](#c-bindings-strategy)
9. [EasyAntiCheat Compliance](#easyanticheat-compliance)
10. [Build System](#build-system)
11. [Testing Strategy](#testing-strategy)
12. [Module Specifications](#module-specifications)

---

## Overview

DaktLib is a collection of **standalone, modular libraries** designed for building third-party utility applications for Star Citizen. Each module is:

- **100% dependency-free** (no external runtime dependencies for licensing purity)
- **Independently compilable** (each module is a separate crate)
- **Cross-platform** (Windows, Linux, macOS)
- **C# bindable** (FFI exports with generated C# wrappers)
- **Plug-and-play** (modules communicate via traits, not hard dependencies)

### Target Applications

- Inventory management tools
- Trade route calculators
- Mining/salvage overlays
- Ship loadout managers
- Organization management tools
- Game log analyzers
- Screen-based OCR utilities

---

## Design Principles

### 1. Zero External Dependencies
```
┌─────────────────────────────────────────────────────────┐
│                    DaktLib Modules                       │
│  ┌─────────────────────────────────────────────────────┐│
│  │  No: Dear ImGui, SQLite bindings, curl, etc.        ││
│  │  Yes: Pure Rust implementations, OS APIs only       ││
│  └─────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────┘
```

### 2. Trait-Based Abstraction
Every module defines traits for its core functionality, allowing:
- Custom implementations by consumers
- Easy mocking for tests
- Swappable backends

### 3. Feature-Gated Compilation
```toml
[features]
default = []
std = []           # Standard library support
alloc = []         # Heap allocation only
logging = []       # Enable logging trait integration
async = []         # Async runtime support
csharp-ffi = []    # Generate C# bindings
```

### 4. Minimal Public API Surface
- Expose only what's necessary
- Internal implementation details stay private
- Semantic versioning with careful API evolution

### 5. EAC-Safe Design
- No process injection or memory manipulation
- External-only interactions with game
- Screen capture via OS APIs only

---

## Language & Tooling

### Why Rust?

| Requirement | Rust Solution |
|-------------|---------------|
| Cross-platform | Native tier-1 support for Windows, Linux, macOS |
| Dependency-free | `#![no_std]` support, feature flags |
| C# Bindings | `csbindgen`, `interoptopus`, `uniffi` |
| Memory Safety | Compile-time borrow checker |
| Performance | Zero-cost abstractions |
| Modularity | Cargo workspaces + features |
| ONNX Support | `ort` crate (optional linking) |

### Toolchain Requirements

```toml
# rust-toolchain.toml
[toolchain]
channel = "stable"
components = ["rustfmt", "clippy", "rust-src"]
targets = [
    "x86_64-pc-windows-msvc",
    "x86_64-unknown-linux-gnu",
    "x86_64-apple-darwin",
    "aarch64-apple-darwin"
]
```

### Build Tools

- **Cargo** - Rust package manager and build system
- **cargo-make** - Task runner for complex builds
- **csbindgen** - C# binding generation
- **cargo-nextest** - Fast test runner
- **cargo-deny** - License and dependency auditing

---

## Workspace Structure

```
DaktLib/
├── Cargo.toml              # Workspace root
├── ARCHITECTURE.md         # This document
├── TODO.md                 # Development roadmap
├── LICENSE                 # MIT License
├── README.md               # Project overview
│
├── crates/                 # All Rust crates
│   ├── daktlib/            # Meta-crate (re-exports all)
│   │
│   ├── daktlib-logger/     # Logging traits & default impl
│   ├── daktlib-events/     # Event system (pub/sub)
│   ├── daktlib-vfs/        # Virtual file system
│   ├── daktlib-parser/     # Game data parsers
│   ├── daktlib-sql/        # Embedded SQL database
│   ├── daktlib-http/       # HTTP client
│   ├── daktlib-gui/        # Custom GUI framework
│   ├── daktlib-overlay/    # Screen overlay system
│   ├── daktlib-ocr/        # OCR engine
│   ├── daktlib-ml/         # ML/ONNX inference
│   ├── daktlib-export/     # Data export formats
│   └── daktlib-capture/    # Screen capture
│
├── bindings/               # Language bindings
│   └── csharp/
│       ├── DaktLib.Logger/
│       ├── DaktLib.Events/
│       ├── DaktLib.VFS/
│       ├── DaktLib.Parser/
│       ├── DaktLib.SQL/
│       ├── DaktLib.Http/
│       ├── DaktLib.GUI/
│       ├── DaktLib.Overlay/
│       ├── DaktLib.OCR/
│       ├── DaktLib.ML/
│       ├── DaktLib.Export/
│       └── DaktLib.Capture/
│
├── examples/               # Example applications
│   ├── rust/
│   └── csharp/
│
└── docs/                   # Additional documentation
    ├── api/
    └── tutorials/
```

### Current Folder Mapping

| Current Folder | New Crate Location |
|----------------|-------------------|
| `Events/` | `crates/daktlib-events/` |
| `Export/` | `crates/daktlib-export/` |
| `GUI/` | `crates/daktlib-gui/` |
| `Http/` | `crates/daktlib-http/` |
| `Logger/` | `crates/daktlib-logger/` |
| `OCR/` | `crates/daktlib-ocr/` |
| `Overlay/` | `crates/daktlib-overlay/` |
| `Parser/` | `crates/daktlib-parser/` |
| `SQL/` | `crates/daktlib-sql/` |
| `VFS/` | `crates/daktlib-vfs/` |
| *(new)* | `crates/daktlib-ml/` |
| *(new)* | `crates/daktlib-capture/` |

---

## Module Dependency Graph

```
                    ┌─────────────────┐
                    │  daktlib-logger │ (TRAIT DEFINITIONS ONLY)
                    │   LoggerTrait   │
                    └────────┬────────┘
                             │ (optional trait impl)
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│ daktlib-events│   │  daktlib-vfs  │   │ daktlib-http  │
│  EventBusTrait│   │   VfsTrait    │   │  HttpTrait    │
└───────┬───────┘   └───────┬───────┘   └───────┬───────┘
        │                   │                   │
        │           ┌───────┴───────┐           │
        │           ▼               ▼           │
        │   ┌───────────────┐ ┌───────────────┐ │
        │   │daktlib-parser │ │  daktlib-sql  │ │
        │   │  ParserTrait  │ │   SqlTrait    │ │
        │   └───────────────┘ └───────────────┘ │
        │                                       │
        └───────────────┬───────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
┌───────────────┐ ┌───────────────┐ ┌───────────────┐
│  daktlib-gui  │ │daktlib-overlay│ │ daktlib-export│
│   GuiTrait    │ │ OverlayTrait  │ │  ExportTrait  │
└───────────────┘ └───────┬───────┘ └───────────────┘
                          │
                ┌─────────┴─────────┐
                ▼                   ▼
        ┌───────────────┐   ┌───────────────┐
        │daktlib-capture│   │  daktlib-ocr  │
        │ CaptureTrait  │   │   OcrTrait    │
        └───────────────┘   └───────┬───────┘
                                    │
                                    ▼
                            ┌───────────────┐
                            │  daktlib-ml   │
                            │   MlTrait     │
                            └───────────────┘
```

### Key Points:

1. **No hard dependencies** - Arrows represent *optional trait consumption*
2. **Logger is universal** - Every module can optionally accept a `Logger` trait impl
3. **Overlay + Capture + OCR** - Form the screen analysis pipeline
4. **ML is leaf** - Used only by OCR for ONNX inference

---

## Plug-and-Play Architecture

### Trait Definition Pattern

Every module follows this pattern:

```rust
// In daktlib-logger/src/lib.rs

/// The core logging trait - implement this to provide custom logging
pub trait Logger: Send + Sync + 'static {
    /// Log a message at the given level
    fn log(&self, level: LogLevel, target: &str, message: &str);
    
    /// Flush any buffered logs
    fn flush(&self);
    
    /// Check if a level is enabled
    fn enabled(&self, level: LogLevel) -> bool;
}

/// Log levels supported by the logger
#[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord)]
#[repr(C)]  // For FFI compatibility
pub enum LogLevel {
    Trace = 0,
    Debug = 1,
    Info = 2,
    Warn = 3,
    Error = 4,
}

/// A no-op logger for when logging is disabled
pub struct NullLogger;

impl Logger for NullLogger {
    fn log(&self, _: LogLevel, _: &str, _: &str) {}
    fn flush(&self) {}
    fn enabled(&self, _: LogLevel) -> bool { false }
}

/// Default file/console logger (optional feature)
#[cfg(feature = "default-logger")]
pub struct DefaultLogger { /* ... */ }
```

### Consumer Pattern

```rust
// In daktlib-ocr/src/lib.rs

use daktlib_logger::{Logger, NullLogger};

pub struct OcrEngine<L: Logger = NullLogger> {
    logger: L,
    // ... other fields
}

impl<L: Logger> OcrEngine<L> {
    pub fn new(logger: L) -> Self {
        Self { logger }
    }
    
    pub fn recognize(&self, image: &[u8]) -> Result<String, OcrError> {
        self.logger.log(LogLevel::Debug, "ocr", "Starting recognition");
        // ... implementation
    }
}

// Convenience constructor with no logging
impl OcrEngine<NullLogger> {
    pub fn without_logging() -> Self {
        Self::new(NullLogger)
    }
}
```

### Builder Pattern for Complex Modules

```rust
pub struct OverlayBuilder<L: Logger = NullLogger, C: Capture = NullCapture> {
    logger: Option<L>,
    capture: Option<C>,
    // ... configuration
}

impl OverlayBuilder {
    pub fn new() -> Self { /* ... */ }
    
    pub fn with_logger<L: Logger>(self, logger: L) -> OverlayBuilder<L, _> {
        // ...
    }
    
    pub fn with_capture<C: Capture>(self, capture: C) -> OverlayBuilder<_, C> {
        // ...
    }
    
    pub fn build(self) -> Result<Overlay<L, C>, BuildError> {
        // ...
    }
}
```

---

## Cross-Cutting Concerns

### Error Handling

Each module defines its own error type that implements standard traits:

```rust
// daktlib-error-derive macro (internal)
#[derive(Debug, Clone)]
pub enum OcrError {
    InvalidImage { reason: String },
    ModelNotLoaded,
    InferenceFailed { code: i32 },
    // ...
}

impl std::fmt::Display for OcrError { /* ... */ }
impl std::error::Error for OcrError { /* ... */ }

// FFI-safe error representation
#[repr(C)]
pub struct FfiError {
    pub code: i32,
    pub message: *const c_char,
}
```

### Configuration

```rust
// Each module has a Config struct
#[derive(Debug, Clone, Default)]
#[cfg_attr(feature = "serde", derive(Serialize, Deserialize))]
pub struct OcrConfig {
    pub model_path: Option<PathBuf>,
    pub language: String,
    pub confidence_threshold: f32,
}
```

### Async Support

```rust
// Gated behind "async" feature
#[cfg(feature = "async")]
pub trait AsyncLogger: Send + Sync + 'static {
    fn log(&self, level: LogLevel, target: &str, message: &str) 
        -> impl Future<Output = ()> + Send;
}
```

---

## C# Bindings Strategy

### Generation Approach

Using `csbindgen` for automatic generation:

```rust
// In each module's ffi.rs
use csbindgen::*;

#[no_mangle]
pub extern "C" fn daktlib_logger_create() -> *mut Logger {
    Box::into_raw(Box::new(DefaultLogger::new()))
}

#[no_mangle]
pub extern "C" fn daktlib_logger_log(
    logger: *mut Logger,
    level: LogLevel,
    message: *const c_char,
) {
    // ... safe wrapper
}

#[no_mangle]
pub extern "C" fn daktlib_logger_destroy(logger: *mut Logger) {
    if !logger.is_null() {
        unsafe { drop(Box::from_raw(logger)); }
    }
}
```

### C# Wrapper Structure

```csharp
// DaktLib.Logger/Logger.cs
namespace DaktLib.Logger
{
    public class Logger : IDisposable
    {
        private IntPtr _handle;
        
        public Logger()
        {
            _handle = NativeMethods.daktlib_logger_create();
        }
        
        public void Log(LogLevel level, string message)
        {
            NativeMethods.daktlib_logger_log(_handle, level, message);
        }
        
        public void Dispose()
        {
            if (_handle != IntPtr.Zero)
            {
                NativeMethods.daktlib_logger_destroy(_handle);
                _handle = IntPtr.Zero;
            }
        }
    }
    
    // Interface for custom implementations
    public interface ILogger
    {
        void Log(LogLevel level, string message);
        void Flush();
    }
}
```

### Build Integration

```xml
<!-- DaktLib.Logger.csproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
  </PropertyGroup>
  
  <ItemGroup>
    <NativeLibrary Include="../../../target/release/daktlib_logger.dll" 
                   Condition="'$(OS)' == 'Windows_NT'" />
    <NativeLibrary Include="../../../target/release/libdaktlib_logger.so" 
                   Condition="'$(OS)' == 'Unix'" />
  </ItemGroup>
</Project>
```

---

## EasyAntiCheat Compliance

### Allowed Techniques ✅

| Technique | Module | Implementation |
|-----------|--------|----------------|
| Screen Capture | `daktlib-capture` | DXGI Desktop Duplication, BitBlt |
| Overlay Windows | `daktlib-overlay` | Transparent WS_EX_TOPMOST windows |
| Log File Parsing | `daktlib-parser` | Read Game.log externally |
| HTTP API Calls | `daktlib-http` | RSI API, community APIs |
| OCR on Screenshots | `daktlib-ocr` | Process captured frames |

### Prohibited Techniques ❌

| Technique | Why Prohibited |
|-----------|----------------|
| ReadProcessMemory | Direct game memory access |
| WriteProcessMemory | Game memory modification |
| DLL Injection | Code injection into game |
| Import/Export Hooking | Function interception |
| Direct3D Hooking | Graphics API interception |
| Kernel Drivers | Ring-0 access |

### Implementation Guidelines

```rust
// daktlib-capture: DXGI Desktop Duplication (Windows)
pub struct DesktopDuplication {
    // Uses IDXGIOutputDuplication - captures entire desktop
    // No game process interaction
}

// daktlib-overlay: External Window (Windows)
pub struct OverlayWindow {
    // Creates WS_EX_TOPMOST | WS_EX_TRANSPARENT | WS_EX_LAYERED window
    // Renders with Direct2D or GDI+ to our own window
    // No injection into game process
}
```

---

## Build System

### Cargo Workspace Configuration

```toml
# Cargo.toml (root)
[workspace]
resolver = "2"
members = [
    "crates/daktlib",
    "crates/daktlib-logger",
    "crates/daktlib-events",
    "crates/daktlib-vfs",
    "crates/daktlib-parser",
    "crates/daktlib-sql",
    "crates/daktlib-http",
    "crates/daktlib-gui",
    "crates/daktlib-overlay",
    "crates/daktlib-ocr",
    "crates/daktlib-ml",
    "crates/daktlib-export",
    "crates/daktlib-capture",
]

[workspace.package]
version = "0.1.0"
edition = "2021"
rust-version = "1.75"
license = "MIT"
repository = "https://github.com/yourusername/DaktLib"
authors = ["Your Name <your.email@example.com>"]

[workspace.dependencies]
# Internal dependencies (version = workspace)
daktlib-logger = { path = "crates/daktlib-logger" }
daktlib-events = { path = "crates/daktlib-events" }
# ... etc

[workspace.lints.rust]
unsafe_code = "warn"
missing_docs = "warn"

[workspace.lints.clippy]
all = "warn"
pedantic = "warn"
```

### Makefile.toml (cargo-make)

```toml
[tasks.build-all]
script = "cargo build --release --workspace"

[tasks.build-csharp-bindings]
dependencies = ["build-all"]
script = """
cd bindings/csharp
dotnet build -c Release
"""

[tasks.test-all]
script = "cargo nextest run --workspace"

[tasks.audit]
script = "cargo deny check"

[tasks.docs]
script = "cargo doc --workspace --no-deps"
```

---

## Testing Strategy

### Unit Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_logger_levels() {
        let logger = DefaultLogger::new();
        assert!(logger.enabled(LogLevel::Error));
    }
}
```

### Integration Tests

```rust
// tests/integration/logger_ocr.rs
#[test]
fn test_ocr_with_custom_logger() {
    struct TestLogger(Vec<String>);
    impl Logger for TestLogger { /* ... */ }
    
    let logger = TestLogger(vec![]);
    let ocr = OcrEngine::new(logger);
    // ...
}
```

### Property-Based Tests

```rust
#[cfg(test)]
mod proptests {
    use proptest::prelude::*;
    
    proptest! {
        #[test]
        fn parser_roundtrip(data: Vec<u8>) {
            // ...
        }
    }
}
```

### FFI Tests (C#)

```csharp
[Test]
public void TestLoggerCreation()
{
    using var logger = new Logger();
    logger.Log(LogLevel.Info, "Test message");
    // No crash = success
}
```

---

## Module Specifications

### Core Modules

| Module | Purpose | Key Traits |
|--------|---------|------------|
| `daktlib-logger` | Logging abstraction | `Logger`, `AsyncLogger` |
| `daktlib-events` | Event bus / pub-sub | `EventBus`, `EventHandler` |
| `daktlib-vfs` | Virtual file system | `Vfs`, `VfsEntry` |
| `daktlib-parser` | Game data parsing | `Parser`, `DataForge` |
| `daktlib-sql` | Embedded database | `Database`, `Query` |
| `daktlib-http` | HTTP client | `HttpClient`, `Request` |

### GUI/Visual Modules

| Module | Purpose | Key Traits |
|--------|---------|------------|
| `daktlib-gui` | Custom GUI framework | `Widget`, `Renderer` |
| `daktlib-overlay` | Screen overlay | `Overlay`, `Layer` |
| `daktlib-capture` | Screen capture | `Capture`, `Frame` |

### ML/AI Modules

| Module | Purpose | Key Traits |
|--------|---------|------------|
| `daktlib-ocr` | Text recognition | `OcrEngine`, `TextRegion` |
| `daktlib-ml` | ONNX inference | `Model`, `Tensor` |

### Utility Modules

| Module | Purpose | Key Traits |
|--------|---------|------------|
| `daktlib-export` | Data export | `Exporter`, `Format` |

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 0.1.0 | 2026-01-03 | Initial architecture design |

---

## References

- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- [csbindgen Documentation](https://github.com/Cysharp/csbindgen)
- [DXGI Desktop Duplication](https://docs.microsoft.com/en-us/windows/win32/direct3ddxgi/desktop-dup-api)
- [ONNX Runtime](https://onnxruntime.ai/)
- [Star Citizen EAC Documentation](https://support.robertsspaceindustries.com/)
