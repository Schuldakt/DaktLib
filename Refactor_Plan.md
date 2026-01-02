# DaktLib Standalone Module Refactoring Plan

## Executive Summary

This document outlines the complete refactoring strategy to make all DaktLib modules **100% standalone** with zero inter-module dependencies, plus the addition of two new modules: **HTTP** and **SQL**.

---

## Current State Analysis

### Existing Inter-Module Dependencies (Problems)

| Module | Currently Depends On | Issue |
|--------|---------------------|-------|
| **Logger** | Core | ✅ Acceptable (Core is foundation) |
| **Events** | Core | ✅ Acceptable |
| **Config** | Core | ✅ Acceptable |
| **VFS** | Core, **Parser** | ❌ Parser dependency for P4K/PAK |
| **Parser** | Core, **Logger**, **VFS** | ❌ Circular with VFS, Logger coupling |
| **GUI** | Core, **Logger**, **Events** | ❌ Logger/Events coupling |
| **Export** | Core | ✅ Acceptable |
| **Overlay** | Core, **Logger**, **GUI** | ❌ Logger/GUI coupling |
| **OCR** | Core, **Logger** | ❌ Logger coupling |

### The Core Problem

If someone wants to use just `VFS` in their project, they currently need to also link:
- Core ✅ (unavoidable, provides types)
- Parser ❌ (unnecessary if not reading .p4k files)
- Logger ❌ (unnecessary if they have their own)

**Goal:** Each module should compile and work with **only** Core (or nothing at all).

---

## Target Architecture

### New Dependency Graph (Zero Inter-Module Dependencies)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STANDALONE MODULES                               │
│         (Each compiles independently, no inter-module deps)         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐           │
│   │  Core  │ │ Logger │ │ Events │ │ Config │ │  HTTP  │ ← NEW     │
│   └────────┘ └────────┘ └────────┘ └────────┘ └────────┘           │
│                                                                     │
│   ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐           │
│   │  VFS   │ │ Parser │ │  GUI   │ │ Export │ │  SQL   │ ← NEW     │
│   └────────┘ └────────┘ └────────┘ └────────┘ └────────┘           │
│                                                                     │
│   ┌────────┐ ┌────────┐                                             │
│   │Overlay │ │  OCR   │                                             │
│   └────────┘ └────────┘                                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    INTEGRATION LAYER (Optional)                     │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    DaktLib-Integration                      │   │
│   │  Glue code that wires modules together for DaktExplorer/    │   │
│   │  DaktOverlay. NOT required for standalone module use.       │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Refactoring Strategies

### Strategy 1: Interface Injection (Primary Approach)

Replace hard dependencies with **interface callbacks** that users can optionally provide.

**Example - VFS with optional archive support:**

```cpp
// Before (hard dependency on Parser)
#include <dakt/parser/CryPakParser.hpp>
class P4kProvider : public IFileProvider {
    CryPakParser m_parser;  // Hard dependency!
};

// After (interface injection)
class IArchiveHandler {
public:
    virtual ~IArchiveHandler() = default;
    virtual Result<std::vector<std::string>> listFiles() = 0;
    virtual Result<std::vector<uint8_t>> readFile(std::string_view path) = 0;
};

class ArchiveProvider : public IFileProvider {
public:
    explicit ArchiveProvider(std::unique_ptr<IArchiveHandler> handler);
    // Works with ANY archive format - ZIP, P4K, PAK, etc.
};
```

**Usage in application:**
```cpp
// In DaktExplorer (has Parser available)
#include <dakt/vfs/VFS.hpp>
#include <dakt/parser/CryPakParser.hpp>
#include <dakt/integration/P4kArchiveHandler.hpp>  // Glue code

auto handler = std::make_unique<P4kArchiveHandler>(path);  // Uses Parser
vfs.mount("/data", std::make_unique<ArchiveProvider>(std::move(handler)));

// In standalone project (no Parser)
#include <dakt/vfs/VFS.hpp>
#include "MyZipHandler.hpp"  // User's own implementation

auto handler = std::make_unique<MyZipHandler>(path);
vfs.mount("/data", std::make_unique<ArchiveProvider>(std::move(handler)));
```

### Strategy 2: Compile-Time Feature Flags

Use CMake options to conditionally include integration code.

```cmake
# CMakeLists.txt
option(DAKT_VFS_P4K_SUPPORT "Enable P4K archive support (requires Parser)" OFF)
option(DAKT_GUI_LOGGING "Enable GUI logging (requires Logger)" OFF)
option(DAKT_OVERLAY_GUI "Enable Overlay GUI (requires GUI)" OFF)

if(DAKT_VFS_P4K_SUPPORT)
    target_compile_definitions(DaktVFS PRIVATE DAKT_HAS_PARSER)
    target_link_libraries(DaktVFS PRIVATE DaktParser)
endif()
```

### Strategy 3: Callback-Based Logging

Replace Logger dependency with a simple callback interface.

```cpp
// Universal log callback (defined in each module independently)
namespace dakt::vfs {
    using LogCallback = std::function<void(int level, std::string_view message)>;
    
    inline LogCallback g_logCallback = nullptr;
    
    inline void setLogCallback(LogCallback cb) { g_logCallback = std::move(cb); }
    
    // Internal macro
    #define VFS_LOG(level, ...) \
        if (g_logCallback) g_logCallback(level, std::format(__VA_ARGS__))
}

// User wires it up if they want logging
#include <dakt/vfs/VFS.hpp>
#include <dakt/logger/Logger.hpp>

dakt::vfs::setLogCallback([](int level, std::string_view msg) {
    DAKT_LOG(static_cast<LogLevel>(level), "{}", msg);
});
```

---

## Module-by-Module Refactoring Plan

### Phase 1: Foundation Modules (No Changes Needed)

| Module | Current Deps | Action |
|--------|--------------|--------|
| **Core** | C++ stdlib only | ✅ Already standalone |
| **Logger** | Core types only | ✅ Already standalone |
| **Events** | Core types only | ✅ Already standalone |
| **Config** | Core types only | ✅ Already standalone |

### Phase 2: New Modules

#### HTTP Module (New)

**Architecture:**
```
HTTP/
├── include/dakt/http/
│   ├── HTTP.hpp              # Main convenience header
│   ├── Types.hpp             # Request, Response, Headers, Status codes
│   ├── Client.hpp            # HTTP client (sync + async)
│   ├── Request.hpp           # Request builder
│   ├── Response.hpp          # Response parser
│   ├── URL.hpp               # URL parsing/building
│   ├── Headers.hpp           # Header manipulation
│   ├── Cookie.hpp            # Cookie jar
│   ├── Auth.hpp              # Basic, Bearer, OAuth helpers
│   ├── WebSocket.hpp         # WebSocket client
│   └── backends/
│       ├── IHttpBackend.hpp  # Abstract backend interface
│       ├── WinHttpBackend.hpp # Windows (WinHTTP)
│       ├── CurlBackend.hpp   # Cross-platform (optional libcurl)
│       └── SocketBackend.hpp # Raw socket implementation (no deps)
└── src/
    ├── URL.cpp
    ├── Headers.cpp
    ├── Request.cpp
    ├── Response.cpp
    ├── Client.cpp
    ├── WebSocket.cpp
    └── backends/
        ├── WinHttpBackend.cpp
        └── SocketBackend.cpp
```

**Key Design Decisions:**
- **Primary backend:** Raw socket implementation (truly zero dependencies)
- **Optional backends:** WinHTTP (Windows), libcurl (if user has it)
- **Features:** GET/POST/PUT/DELETE, headers, cookies, redirects, timeouts, async
- **TLS:** Optional - raw sockets = HTTP only; WinHTTP/schannel = HTTPS on Windows

**API Preview:**
```cpp
#include <dakt/http/HTTP.hpp>

// Simple GET
auto response = Http::get("http://api.example.com/data");
if (response) {
    std::cout << response->body();
}

// POST with JSON
auto response = Http::post("http://api.example.com/submit")
    .header("Content-Type", "application/json")
    .body(R"({"name": "test"})")
    .timeout(std::chrono::seconds(30))
    .send();

// Async
Http::getAsync("http://api.example.com/data")
    .then([](Response r) { process(r); })
    .catchError([](Error e) { handle(e); });
```

---

#### SQL Module (New)

**Architecture:**
```
SQL/
├── include/dakt/sql/
│   ├── SQL.hpp               # Main convenience header
│   ├── Types.hpp             # Value, Row, Result types
│   ├── Database.hpp          # Database connection
│   ├── Statement.hpp         # Prepared statement
│   ├── Query.hpp             # Query builder (optional fluent API)
│   ├── Transaction.hpp       # Transaction RAII wrapper
│   ├── Migration.hpp         # Schema migration helpers
│   ├── Pool.hpp              # Connection pooling
│   └── backends/
│       ├── IDatabase.hpp     # Abstract database interface
│       └── SQLite.hpp        # SQLite implementation (built-in)
└── src/
    ├── Database.cpp
    ├── Statement.cpp
    ├── Query.cpp
    ├── Transaction.cpp
    ├── Migration.cpp
    ├── Pool.cpp
    └── backends/
        └── SQLite.cpp        # Full SQLite amalgamation embedded
```

**Key Design Decisions:**
- **SQLite amalgamation embedded** - Single sqlite3.c file compiled in, truly zero dependencies
- **Prepared statements** - Prevent SQL injection, better performance
- **RAII transactions** - Auto-rollback on exception/scope exit
- **Type-safe binding** - Concepts for bindable types
- **Optional query builder** - Fluent API for dynamic queries

**API Preview:**
```cpp
#include <dakt/sql/SQL.hpp>

// Open database (creates if not exists)
Database db("app.db");

// Create table
db.execute(R"(
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY,
        name TEXT NOT NULL,
        email TEXT UNIQUE
    )
)");

// Prepared statement with binding
auto stmt = db.prepare("INSERT INTO users (name, email) VALUES (?, ?)");
stmt.bind(1, "John");
stmt.bind(2, "john@example.com");
stmt.execute();

// Query with results
auto results = db.query("SELECT * FROM users WHERE name LIKE ?", "%John%");
for (const auto& row : results) {
    std::cout << row.get<std::string>("name") << "\n";
}

// Transaction
{
    Transaction tx(db);
    db.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1");
    db.execute("UPDATE accounts SET balance = balance + 100 WHERE id = 2");
    tx.commit();  // Auto-rollback if not committed
}

// Query builder (optional)
auto users = Query(db)
    .select("name", "email")
    .from("users")
    .where("active", true)
    .orderBy("name")
    .limit(10)
    .fetch();
```

---

### Phase 3: Refactor Existing Modules

#### VFS Refactoring

**Changes Required:**
1. Remove `#include <dakt/parser/CryPakParser.hpp>`
2. Replace `P4kProvider` and `PakProvider` with generic `ArchiveProvider`
3. Add `IArchiveHandler` interface
4. Move P4K/PAK specific code to Integration layer

**New Structure:**
```
VFS/
├── include/dakt/vfs/
│   ├── VFS.hpp
│   ├── IFileProvider.hpp
│   ├── IArchiveHandler.hpp     # NEW - Interface for archive formats
│   ├── PhysicalProvider.hpp
│   ├── MemoryProvider.hpp
│   ├── ArchiveProvider.hpp     # NEW - Generic archive provider
│   ├── OverlayProvider.hpp
│   └── ...
└── src/
    └── ...
```

**Removed from VFS (moved to Integration):**
- `P4kProvider.hpp/.cpp`
- `PakProvider.hpp/.cpp`

---

#### Parser Refactoring

**Changes Required:**
1. Remove Logger dependency → Use callback-based logging
2. Remove VFS dependency → Accept `std::istream` or `std::span<uint8_t>` instead

**Before:**
```cpp
Result<DataCore> DataCoreParser::parse(const VirtualFile& file);
```

**After:**
```cpp
Result<DataCore> DataCoreParser::parse(std::span<const uint8_t> data);
Result<DataCore> DataCoreParser::parse(std::istream& stream);
```

---

#### GUI Refactoring

**Changes Required:**
1. Remove Logger dependency → Callback-based logging
2. Remove Events dependency → Built-in simple signal/slot OR callback interface

**Options for Events replacement:**
- **Option A:** Inline a minimal signal implementation (adds ~100 lines)
- **Option B:** Use raw callbacks (`std::function`) for widget events
- **Option C:** Keep Events as an **optional** compile-time dependency

**Recommended: Option B** - Raw callbacks are simpler and don't require Events module.

```cpp
// Before (Events dependency)
Button btn;
btn.onClick.connect([](){ doSomething(); });

// After (standalone)
Button btn;
btn.setOnClick([](){ doSomething(); });
```

---

#### Overlay Refactoring

**Changes Required:**
1. Remove Logger dependency → Callback-based logging
2. Remove GUI dependency → Accept `IRenderContext` interface

**New Interface:**
```cpp
class IRenderContext {
public:
    virtual void drawRect(Rect r, Color c) = 0;
    virtual void drawText(Vec2 pos, std::string_view text, Color c) = 0;
    // ...
};

class OverlayWindow {
public:
    void setRenderContext(std::unique_ptr<IRenderContext> ctx);
};
```

**In Integration layer:**
```cpp
// DaktGuiRenderContext implements IRenderContext using DaktGui
class DaktGuiRenderContext : public IRenderContext { ... };

overlay.setRenderContext(std::make_unique<DaktGuiRenderContext>(guiContext));
```

---

#### OCR Refactoring

**Changes Required:**
1. Remove Logger dependency → Callback-based logging

(OCR is already mostly standalone, just needs logging decoupled)

---

### Phase 4: Integration Layer

Create a new `Integration/` module that wires everything together for DaktLib applications.

```
Integration/
├── include/dakt/integration/
│   ├── P4kArchiveHandler.hpp     # IArchiveHandler for P4K using Parser
│   ├── PakArchiveHandler.hpp     # IArchiveHandler for PAK using Parser
│   ├── DaktGuiRenderContext.hpp  # IRenderContext using GUI module
│   ├── LoggerBridge.hpp          # Connects module log callbacks to Logger
│   └── EventBridge.hpp           # Connects module events to Events system
└── src/
    ├── P4kArchiveHandler.cpp
    ├── PakArchiveHandler.cpp
    ├── DaktGuiRenderContext.cpp
    ├── LoggerBridge.cpp
    └── EventBridge.cpp
```

**Usage in DaktExplorer:**
```cpp
#include <dakt/vfs/VFS.hpp>
#include <dakt/gui/GUI.hpp>
#include <dakt/parser/Parser.hpp>
#include <dakt/logger/Logger.hpp>
#include <dakt/integration/Integration.hpp>  // Glue code

int main() {
    // Wire up logging for all modules
    Integration::setupLogging();
    
    // Create VFS with P4K support
    VirtualFileSystem vfs;
    auto p4kHandler = Integration::createP4kHandler("Data.p4k");
    vfs.mount("/data", std::make_unique<ArchiveProvider>(std::move(p4kHandler)));
    
    // Create GUI
    auto gui = GUI::create();
    
    // Create overlay with GUI rendering
    auto overlay = Overlay::create();
    overlay.setRenderContext(Integration::createGuiRenderContext(gui));
    
    // ...
}
```

---

## Implementation Roadmap

### Week 1: Core Infrastructure

| Day | Task |
|-----|------|
| 1-2 | Create callback-based logging infrastructure for all modules |
| 3-4 | Define `IArchiveHandler` interface, create `ArchiveProvider` |
| 5 | Define `IRenderContext` interface |

### Week 2: New Modules - HTTP

| Day | Task |
|-----|------|
| 1 | HTTP Types, URL parser |
| 2-3 | Raw socket backend (HTTP/1.1) |
| 4 | WinHTTP backend (HTTPS support) |
| 5 | Client API, async support |

### Week 3: New Modules - SQL

| Day | Task |
|-----|------|
| 1 | Embed SQLite amalgamation, basic wrapper |
| 2 | Prepared statements, type-safe binding |
| 3 | Transaction support, RAII wrapper |
| 4 | Query builder (optional fluent API) |
| 5 | Connection pooling, migration helpers |

### Week 4: Module Refactoring

| Day | Task |
|-----|------|
| 1 | Refactor VFS (remove Parser dependency) |
| 2 | Refactor Parser (remove Logger/VFS dependency) |
| 3 | Refactor GUI (remove Logger/Events dependency) |
| 4 | Refactor Overlay (remove Logger/GUI dependency) |
| 5 | Refactor OCR (remove Logger dependency) |

### Week 5: Integration Layer

| Day | Task |
|-----|------|
| 1-2 | Create Integration module with archive handlers |
| 3 | Create render context adapters |
| 4 | Create logger/event bridges |
| 5 | Update DaktExplorer to use Integration layer |

### Week 6: Testing & Documentation

| Day | Task |
|-----|------|
| 1-3 | Write unit tests for new modules |
| 4-5 | Update all documentation, create usage examples |

---

## File Changes Summary

### New Files

```
DaktLib/
├── HTTP/                           # NEW MODULE
│   ├── CMakeLists.txt
│   ├── include/dakt/http/...
│   └── src/...
│
├── SQL/                            # NEW MODULE
│   ├── CMakeLists.txt
│   ├── include/dakt/sql/...
│   ├── src/...
│   └── third_party/
│       └── sqlite3.c              # SQLite amalgamation
│
├── Integration/                    # NEW MODULE
│   ├── CMakeLists.txt
│   ├── include/dakt/integration/...
│   └── src/...
```

### Modified Files

```
VFS/
├── include/dakt/vfs/
│   ├── IArchiveHandler.hpp        # NEW
│   ├── ArchiveProvider.hpp        # NEW (replaces P4k/PakProvider)
│   ├── P4kProvider.hpp            # DELETED (moved to Integration)
│   └── PakProvider.hpp            # DELETED (moved to Integration)

Parser/
├── include/dakt/parser/
│   └── *.hpp                      # Modified to accept span/stream instead of VirtualFile

GUI/
├── include/dakt/gui/
│   └── *.hpp                      # Modified to use callbacks instead of Events

Overlay/
├── include/dakt/overlay/
│   ├── IRenderContext.hpp         # NEW
│   └── *.hpp                      # Modified to use IRenderContext

All Modules:
└── */include/dakt/*/Log.hpp       # NEW - Per-module log callback setup
```

---

## CMake Structure

### Root CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.25)
project(DaktLib VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# All modules can be built independently
option(DAKT_BUILD_CORE "Build Core module" ON)
option(DAKT_BUILD_LOGGER "Build Logger module" ON)
option(DAKT_BUILD_EVENTS "Build Events module" ON)
option(DAKT_BUILD_CONFIG "Build Config module" ON)
option(DAKT_BUILD_HTTP "Build HTTP module" ON)
option(DAKT_BUILD_SQL "Build SQL module" ON)
option(DAKT_BUILD_VFS "Build VFS module" ON)
option(DAKT_BUILD_PARSER "Build Parser module" ON)
option(DAKT_BUILD_GUI "Build GUI module" ON)
option(DAKT_BUILD_EXPORT "Build Export module" ON)
option(DAKT_BUILD_OVERLAY "Build Overlay module" ON)
option(DAKT_BUILD_OCR "Build OCR module" ON)

# Integration layer (requires multiple modules)
option(DAKT_BUILD_INTEGRATION "Build Integration module" ON)

# Applications
option(DAKT_BUILD_EXPLORER "Build DaktExplorer application" ON)
option(DAKT_BUILD_OVERLAY_APP "Build DaktOverlay application" ON)

# Add modules
if(DAKT_BUILD_CORE)
    add_subdirectory(Core)
endif()
# ... etc for each module

# Integration requires specific modules
if(DAKT_BUILD_INTEGRATION)
    if(NOT DAKT_BUILD_VFS OR NOT DAKT_BUILD_PARSER OR NOT DAKT_BUILD_GUI)
        message(WARNING "Integration module requires VFS, Parser, and GUI")
    else()
        add_subdirectory(Integration)
    endif()
endif()
```

### Per-Module CMakeLists.txt (Example: HTTP)

```cmake
cmake_minimum_required(VERSION 3.25)
project(DaktHTTP VERSION 1.0.0 LANGUAGES CXX)

# Can be built completely standalone
add_library(DaktHTTP STATIC
    src/URL.cpp
    src/Headers.cpp
    src/Request.cpp
    src/Response.cpp
    src/Client.cpp
    src/WebSocket.cpp
    src/backends/SocketBackend.cpp
)

target_include_directories(DaktHTTP PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

target_compile_features(DaktHTTP PUBLIC cxx_std_23)

# Optional WinHTTP backend (Windows only)
if(WIN32)
    target_sources(DaktHTTP PRIVATE src/backends/WinHttpBackend.cpp)
    target_link_libraries(DaktHTTP PRIVATE winhttp)
    target_compile_definitions(DaktHTTP PRIVATE DAKT_HTTP_HAS_WINHTTP)
endif()

# No dependencies on other DaktLib modules!
```

---

## Validation Checklist

After refactoring, each module must pass this test:

### Standalone Compilation Test

```bash
# Test: Can HTTP module compile without ANY other DaktLib modules?
mkdir build-http-only && cd build-http-only
cmake ../DaktLib/HTTP -DCMAKE_CXX_STANDARD=23
cmake --build .
# Must succeed with zero errors

# Test: Can SQL module compile without ANY other DaktLib modules?
mkdir build-sql-only && cd build-sql-only
cmake ../DaktLib/SQL -DCMAKE_CXX_STANDARD=23
cmake --build .
# Must succeed with zero errors

# Repeat for ALL modules
```

### Integration Test

```bash
# Test: Can all modules work together via Integration layer?
mkdir build-full && cd build-full
cmake ../DaktLib -DDAKT_BUILD_INTEGRATION=ON
cmake --build .
# Must succeed

# Run DaktExplorer
./DaktExplorer
# Must work correctly
```

---

## Summary

| Aspect | Before | After |
|--------|--------|-------|
| **Total Modules** | 10 | 12 (+ HTTP, SQL) |
| **Inter-module deps** | 8 connections | 0 connections |
| **Standalone capable** | Core only | ALL modules |
| **Integration** | Implicit | Explicit Integration layer |
| **External deps** | mbedtls, zlib | SQLite amalgamation (embedded) |

This architecture gives you:
- **Maximum reusability** - Use any module in any project
- **Clean separation** - Modules don't know about each other
- **Flexibility** - Integration layer is optional glue code
- **Future-proof** - Easy to add new modules without touching existing ones