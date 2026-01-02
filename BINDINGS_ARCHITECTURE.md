# DaktLib Language Bindings Architecture

## Overview

Each DaktLib module will have three layers:

1. **C++ Core** - Full implementation with modern C++23
2. **C API** - FFI-compatible wrapper for cross-language use
3. **Language Bindings** - Idiomatic wrappers for each target language

This allows:
- Pure C++ projects to use the libraries directly
- C# applications to use P/Invoke bindings
- Future support for Python, Rust, or any FFI-capable language

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         YOUR APPLICATIONS                           │
│                                                                     │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐               │
│   │DaktExplorer │   │ DaktOverlay │   │ Future Apps │               │
│   │   (C#/WPF)  │   │  (C#/WPF)   │   │   (C#)      │               │
│   └─────────────┘   └─────────────┘   └─────────────┘               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    LANGUAGE BINDINGS LAYER                          │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    C# / .NET 9 Bindings                     │   │
│   │  • NuGet packages (DaktLib.VFS, DaktLib.Parser, etc.)       │   │
│   │  • Idiomatic C# API (IDisposable, async/await, LINQ)        │   │
│   │  • SafeHandle for native resource management                │   │
│   │  • Source generators for boilerplate                        │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                 Python Bindings (Future)                    │   │
│   │  • PyPI package                                             │   │
│   │  • ctypes or cffi based                                     │   │
│   │  • Pythonic API                                             │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                              │ FFI (P/Invoke, ctypes, etc.)
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         C API LAYER                                 │
│                   (Shared Libraries / DLLs)                         │
│                                                                     │
│   dakt_vfs.dll    dakt_parser.dll    dakt_gui.dll    dakt_http.dll  │
│   dakt_vfs.so     dakt_parser.so     dakt_gui.so     dakt_http.so   │
│   libdakt_vfs.dylib  ...                                            │
│                                                                     │
│   • extern "C" functions only                                       │
│   • Opaque handle types (void*)                                     │
│   • Error codes (no exceptions across FFI boundary)                 │
│   • Callback function pointers for async/events                     │
│   • ABI-stable interface                                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       C++ CORE LAYER                                │
│                   (Static Libraries)                                │
│                                                                     │
│   DaktVFS.lib     DaktParser.lib     DaktGUI.lib     DaktHTTP.lib   │
│   libDaktVFS.a    libDaktParser.a    libDaktGUI.a    libDaktHTTP.a  │
│                                                                     │
│   • Full C++23 implementation                                       │
│   • Templates, concepts, std::expected                              │
│   • Zero-cost abstractions                                          │
│   • Can be used directly by C++ applications                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Module Structure (Updated)

Each module will have this structure:

```
DaktLib-VFS/
├── include/dakt/vfs/           # C++ headers (existing)
│   ├── VFS.hpp
│   ├── core/
│   ├── interfaces/
│   ├── providers/
│   └── util/
│
├── include/dakt/vfs/c/         # C API headers (NEW)
│   ├── dakt_vfs.h              # Main C API header
│   ├── dakt_vfs_types.h        # C types and error codes
│   └── dakt_vfs_exports.h      # DLL export macros
│
├── src/                        # C++ implementation (existing)
│   ├── core/
│   ├── providers/
│   └── util/
│
├── src/c/                      # C API implementation (NEW)
│   └── dakt_vfs_capi.cpp       # C wrapper implementation
│
├── bindings/                   # Language bindings (NEW)
│   ├── csharp/
│   │   ├── DaktLib.VFS/        # C# project
│   │   │   ├── DaktLib.VFS.csproj
│   │   │   ├── Native/         # P/Invoke declarations
│   │   │   │   ├── NativeMethods.cs
│   │   │   │   └── SafeHandles.cs
│   │   │   ├── VirtualFileSystem.cs
│   │   │   ├── VirtualPath.cs
│   │   │   ├── Providers/
│   │   │   │   ├── IFileProvider.cs
│   │   │   │   ├── PhysicalProvider.cs
│   │   │   │   └── ...
│   │   │   └── ...
│   │   └── DaktLib.VFS.Tests/  # C# unit tests
│   │
│   └── python/                 # Future Python bindings
│       └── dakt_vfs/
│
├── CMakeLists.txt              # Updated for shared lib
└── README.md
```

---

## C API Design Principles

### 1. Opaque Handle Types

```c
// dakt_vfs_types.h

// Opaque handle types (pointers to internal C++ objects)
typedef struct DaktVfs* DaktVfsHandle;
typedef struct DaktVfsProvider* DaktVfsProviderHandle;
typedef struct DaktVfsPath* DaktVfsPathHandle;

// Error codes
typedef enum DaktVfsError {
    DAKT_VFS_OK = 0,
    DAKT_VFS_ERROR_INVALID_PATH = 100,
    DAKT_VFS_ERROR_PATH_NOT_FOUND = 101,
    DAKT_VFS_ERROR_FILE_NOT_FOUND = 400,
    // ... etc
} DaktVfsError;

// Result structure for functions that return data
typedef struct DaktVfsResult {
    DaktVfsError error;
    void* data;
    size_t size;
} DaktVfsResult;
```

### 2. Function Naming Convention

```c
// Pattern: dakt_<module>_<object>_<action>

// VFS creation/destruction
DAKT_VFS_API DaktVfsHandle dakt_vfs_create(void);
DAKT_VFS_API void dakt_vfs_destroy(DaktVfsHandle vfs);

// VFS operations
DAKT_VFS_API DaktVfsError dakt_vfs_mount(
    DaktVfsHandle vfs,
    const char* path,
    DaktVfsProviderHandle provider,
    int priority);

DAKT_VFS_API DaktVfsError dakt_vfs_unmount(
    DaktVfsHandle vfs,
    const char* path);

// File operations
DAKT_VFS_API DaktVfsResult dakt_vfs_read_file(
    DaktVfsHandle vfs,
    const char* path);

DAKT_VFS_API void dakt_vfs_free_result(DaktVfsResult* result);

// Provider creation
DAKT_VFS_API DaktVfsProviderHandle dakt_vfs_provider_physical_create(
    const char* root_path);

DAKT_VFS_API void dakt_vfs_provider_destroy(DaktVfsProviderHandle provider);
```

### 3. Callback Patterns

```c
// Callback function types
typedef void (*DaktVfsFileChangeCallback)(
    const char* path,
    DaktVfsFileEvent event,
    void* user_data);

typedef void (*DaktVfsAsyncReadCallback)(
    DaktVfsResult result,
    void* user_data);

typedef void (*DaktVfsLogCallback)(
    DaktVfsLogLevel level,
    const char* message,
    void* user_data);

// Setting callbacks
DAKT_VFS_API void dakt_vfs_set_log_callback(
    DaktVfsLogCallback callback,
    void* user_data);

DAKT_VFS_API DaktVfsError dakt_vfs_watch(
    DaktVfsHandle vfs,
    const char* path,
    DaktVfsFileChangeCallback callback,
    void* user_data,
    int recursive);
```

### 4. Export Macros

```c
// dakt_vfs_exports.h

#ifdef _WIN32
    #ifdef DAKT_VFS_BUILDING_DLL
        #define DAKT_VFS_API __declspec(dllexport)
    #else
        #define DAKT_VFS_API __declspec(dllimport)
    #endif
#else
    #define DAKT_VFS_API __attribute__((visibility("default")))
#endif
```

---

## C# Bindings Design

### 1. P/Invoke Layer (Low-Level)

```csharp
// NativeMethods.cs
namespace DaktLib.VFS.Native;

internal static partial class NativeMethods
{
    private const string LibraryName = "dakt_vfs";

    [LibraryImport(LibraryName)]
    internal static partial IntPtr dakt_vfs_create();

    [LibraryImport(LibraryName)]
    internal static partial void dakt_vfs_destroy(IntPtr vfs);

    [LibraryImport(LibraryName, StringMarshalling = StringMarshalling.Utf8)]
    internal static partial VfsError dakt_vfs_mount(
        IntPtr vfs,
        string path,
        IntPtr provider,
        int priority);

    [LibraryImport(LibraryName, StringMarshalling = StringMarshalling.Utf8)]
    internal static partial VfsResult dakt_vfs_read_file(
        IntPtr vfs,
        string path);

    [LibraryImport(LibraryName)]
    internal static partial void dakt_vfs_free_result(ref VfsResult result);
}
```

### 2. SafeHandle Wrappers

```csharp
// SafeHandles.cs
namespace DaktLib.VFS.Native;

internal sealed class VfsHandle : SafeHandle
{
    public VfsHandle() : base(IntPtr.Zero, true) { }

    public override bool IsInvalid => handle == IntPtr.Zero;

    protected override bool ReleaseHandle()
    {
        NativeMethods.dakt_vfs_destroy(handle);
        return true;
    }
}

internal sealed class ProviderHandle : SafeHandle
{
    public ProviderHandle() : base(IntPtr.Zero, true) { }

    public override bool IsInvalid => handle == IntPtr.Zero;

    protected override bool ReleaseHandle()
    {
        NativeMethods.dakt_vfs_provider_destroy(handle);
        return true;
    }
}
```

### 3. Idiomatic C# API

```csharp
// VirtualFileSystem.cs
namespace DaktLib.VFS;

/// <summary>
/// Virtual file system with mount point support.
/// </summary>
public sealed class VirtualFileSystem : IDisposable
{
    private readonly VfsHandle _handle;
    private bool _disposed;

    public VirtualFileSystem()
    {
        _handle = new VfsHandle();
        _handle.SetHandle(NativeMethods.dakt_vfs_create());
        
        if (_handle.IsInvalid)
            throw new VfsException("Failed to create VFS instance");
    }

    /// <summary>
    /// Mount a provider at the specified path.
    /// </summary>
    public void Mount(string path, IFileProvider provider, Priority priority = Priority.Normal)
    {
        ThrowIfDisposed();
        ArgumentException.ThrowIfNullOrEmpty(path);
        ArgumentNullException.ThrowIfNull(provider);

        var error = NativeMethods.dakt_vfs_mount(
            _handle.DangerousGetHandle(),
            path,
            provider.Handle.DangerousGetHandle(),
            (int)priority);

        if (error != VfsError.Ok)
            throw new VfsException(error, $"Failed to mount at '{path}'");
    }

    /// <summary>
    /// Read a file's contents.
    /// </summary>
    public byte[] ReadFile(string path)
    {
        ThrowIfDisposed();
        ArgumentException.ThrowIfNullOrEmpty(path);

        var result = NativeMethods.dakt_vfs_read_file(
            _handle.DangerousGetHandle(),
            path);

        try
        {
            if (result.Error != VfsError.Ok)
                throw new VfsException(result.Error, $"Failed to read '{path}'");

            var data = new byte[result.Size];
            Marshal.Copy(result.Data, data, 0, (int)result.Size);
            return data;
        }
        finally
        {
            NativeMethods.dakt_vfs_free_result(ref result);
        }
    }

    /// <summary>
    /// Read a file's contents asynchronously.
    /// </summary>
    public async Task<byte[]> ReadFileAsync(string path, CancellationToken cancellationToken = default)
    {
        ThrowIfDisposed();
        ArgumentException.ThrowIfNullOrEmpty(path);

        return await Task.Run(() => ReadFile(path), cancellationToken);
    }

    /// <summary>
    /// Read a file as a string (UTF-8).
    /// </summary>
    public string ReadFileText(string path)
    {
        var bytes = ReadFile(path);
        return Encoding.UTF8.GetString(bytes);
    }

    /// <summary>
    /// Check if a path exists.
    /// </summary>
    public bool Exists(string path)
    {
        ThrowIfDisposed();
        ArgumentException.ThrowIfNullOrEmpty(path);

        return NativeMethods.dakt_vfs_exists(_handle.DangerousGetHandle(), path);
    }

    /// <summary>
    /// List directory contents.
    /// </summary>
    public IEnumerable<DirectoryEntry> ListDirectory(string path)
    {
        ThrowIfDisposed();
        ArgumentException.ThrowIfNullOrEmpty(path);

        // Implementation using native enumeration
        // ...
    }

    public void Dispose()
    {
        if (_disposed) return;
        _handle.Dispose();
        _disposed = true;
    }

    private void ThrowIfDisposed()
    {
        ObjectDisposedException.ThrowIf(_disposed, this);
    }
}
```

### 4. Provider Implementations

```csharp
// PhysicalProvider.cs
namespace DaktLib.VFS.Providers;

/// <summary>
/// File provider that accesses the physical file system.
/// </summary>
public sealed class PhysicalProvider : IFileProvider
{
    internal ProviderHandle Handle { get; }

    public PhysicalProvider(string rootPath)
    {
        ArgumentException.ThrowIfNullOrEmpty(rootPath);

        Handle = new ProviderHandle();
        Handle.SetHandle(NativeMethods.dakt_vfs_provider_physical_create(rootPath));

        if (Handle.IsInvalid)
            throw new VfsException($"Failed to create physical provider for '{rootPath}'");
    }

    public string RootPath { get; }
    public bool IsWritable => true;

    public void Dispose() => Handle.Dispose();
}

// MemoryProvider.cs
public sealed class MemoryProvider : IFileProvider
{
    internal ProviderHandle Handle { get; }

    public MemoryProvider()
    {
        Handle = new ProviderHandle();
        Handle.SetHandle(NativeMethods.dakt_vfs_provider_memory_create());
    }

    /// <summary>
    /// Add a file to the in-memory storage.
    /// </summary>
    public void AddFile(string path, byte[] data)
    {
        // ...
    }

    /// <summary>
    /// Add a file to the in-memory storage.
    /// </summary>
    public void AddFile(string path, string content)
    {
        AddFile(path, Encoding.UTF8.GetBytes(content));
    }

    public void Dispose() => Handle.Dispose();
}
```

### 5. Exception Handling

```csharp
// VfsException.cs
namespace DaktLib.VFS;

public class VfsException : Exception
{
    public VfsError ErrorCode { get; }

    public VfsException(string message) : base(message)
    {
        ErrorCode = VfsError.UnknownError;
    }

    public VfsException(VfsError error, string message) 
        : base($"{message}: {GetErrorMessage(error)}")
    {
        ErrorCode = error;
    }

    private static string GetErrorMessage(VfsError error) => error switch
    {
        VfsError.Ok => "Success",
        VfsError.InvalidPath => "Invalid path",
        VfsError.PathNotFound => "Path not found",
        VfsError.FileNotFound => "File not found",
        VfsError.FileAccessDenied => "Access denied",
        _ => $"Unknown error ({(int)error})"
    };
}
```

### 6. LINQ Extensions

```csharp
// VfsExtensions.cs
namespace DaktLib.VFS;

public static class VfsExtensions
{
    /// <summary>
    /// Find all files matching a pattern recursively.
    /// </summary>
    public static IEnumerable<string> FindFiles(
        this VirtualFileSystem vfs,
        string basePath,
        string pattern)
    {
        return vfs.ListDirectoryRecursive(basePath)
            .Where(e => e.Type == EntryType.File)
            .Where(e => MatchPattern(e.Name, pattern))
            .Select(e => e.Path);
    }

    /// <summary>
    /// Read all files in a directory.
    /// </summary>
    public static IEnumerable<(string Path, byte[] Data)> ReadAllFiles(
        this VirtualFileSystem vfs,
        string path)
    {
        return vfs.ListDirectory(path)
            .Where(e => e.Type == EntryType.File)
            .Select(e => (e.Path, vfs.ReadFile(e.Path)));
    }

    /// <summary>
    /// Read file as JSON and deserialize.
    /// </summary>
    public static T? ReadJson<T>(this VirtualFileSystem vfs, string path)
    {
        var json = vfs.ReadFileText(path);
        return JsonSerializer.Deserialize<T>(json);
    }

    /// <summary>
    /// Read file as JSON and deserialize asynchronously.
    /// </summary>
    public static async Task<T?> ReadJsonAsync<T>(
        this VirtualFileSystem vfs,
        string path,
        CancellationToken cancellationToken = default)
    {
        var data = await vfs.ReadFileAsync(path, cancellationToken);
        using var stream = new MemoryStream(data);
        return await JsonSerializer.DeserializeAsync<T>(stream, cancellationToken: cancellationToken);
    }
}
```

---

## NuGet Package Structure

```
DaktLib.VFS/
├── DaktLib.VFS.csproj
├── lib/
│   ├── net9.0/
│   │   └── DaktLib.VFS.dll
│   └── net8.0/
│       └── DaktLib.VFS.dll
├── runtimes/
│   ├── win-x64/
│   │   └── native/
│   │       └── dakt_vfs.dll
│   ├── linux-x64/
│   │   └── native/
│   │       └── libdakt_vfs.so
│   └── osx-x64/
│       └── native/
│           └── libdakt_vfs.dylib
└── DaktLib.VFS.nuspec
```

---

## CMake Updates for Shared Library

```cmake
# Option to build shared library for FFI
option(DAKT_VFS_BUILD_SHARED "Build shared library for FFI" ON)
option(DAKT_VFS_BUILD_CAPI "Build C API wrapper" ON)

# Static library (core implementation)
add_library(DaktVFS_static STATIC ${DAKT_VFS_SOURCES})

# Shared library (for FFI)
if(DAKT_VFS_BUILD_SHARED AND DAKT_VFS_BUILD_CAPI)
    set(DAKT_VFS_CAPI_SOURCES
        src/c/dakt_vfs_capi.cpp
    )

    add_library(dakt_vfs SHARED
        ${DAKT_VFS_SOURCES}
        ${DAKT_VFS_CAPI_SOURCES}
    )

    target_compile_definitions(dakt_vfs PRIVATE DAKT_VFS_BUILDING_DLL)
    
    # Set output name without lib prefix on all platforms
    set_target_properties(dakt_vfs PROPERTIES
        PREFIX ""
        OUTPUT_NAME "dakt_vfs"
    )
endif()
```

---

## Implementation Roadmap (Updated)

### Phase 1: Foundation (Current)
1. ✅ VFS Module - C++ Core
2. 🔄 VFS Module - C API
3. 🔄 VFS Module - C# Bindings

### Phase 2: Remaining C++ Modules
4. Parser Module (C++ + C API + C#)
5. GUI Module (C++ + C API + C#)
6. Overlay Module (C++ + C API + C#)
7. OCR Module (C++ + C API + C#)

### Phase 3: New Modules
8. HTTP Module (C++ + C API + C#)
9. SQL Module (C++ + C API + C#)

### Phase 4: Applications (C#)
10. DaktExplorer (C# WPF/Avalonia)
11. DaktOverlay (C# WPF)

---

## Benefits of This Architecture

| Benefit | Description |
|---------|-------------|
| **Performance** | C++ core for heavy lifting (parsing, rendering) |
| **Productivity** | C# for rapid application development |
| **Cross-Platform** | .NET 8/9 runs on Windows, Linux, macOS |
| **Memory Safety** | C# GC handles application-level memory |
| **Hot Reload** | C# hot reload for faster iteration |
| **UI Frameworks** | WPF, Avalonia, MAUI available in C# |
| **Future Proof** | C API allows bindings for any language |
| **Testing** | Easier unit testing in C# with xUnit/NUnit |

---

## Example Application (C#)

```csharp
// DaktExplorer/Program.cs
using DaktLib.VFS;
using DaktLib.VFS.Providers;
using DaktLib.Parser;

var app = new DaktExplorerApp();

// Initialize VFS
using var vfs = new VirtualFileSystem();

// Mount Star Citizen data (using Parser's P4K handler via Integration)
using var p4kProvider = new P4kProvider("C:/StarCitizen/Data.p4k");
vfs.Mount("/data", p4kProvider, Priority.Normal);

// Mount user mods
using var modsProvider = new PhysicalProvider("./mods");
vfs.Mount("/data", modsProvider, Priority.High);

// Now use VFS to explore game files
var files = vfs.ListDirectory("/data/Scripts")
    .Where(e => e.Name.EndsWith(".lua"))
    .ToList();

foreach (var file in files)
{
    Console.WriteLine($"Found: {file.Path}");
}

// Parse a DataCore file
var dcbData = vfs.ReadFile("/data/Game.dcb");
using var dataCore = DataCoreParser.Parse(dcbData);

// Query game data
var ships = dataCore.GetRecords("Ship")
    .Where(s => s.GetString("Manufacturer") == "Anvil")
    .Select(s => new { Name = s.GetString("Name"), Price = s.GetInt("BasePrice") })
    .ToList();
```

---

## Summary

This architecture gives you:

1. **C++ Performance** - Heavy computation in native code
2. **C# Productivity** - Rapid application development
3. **Clean Separation** - C API provides stable ABI boundary
4. **Future Flexibility** - Python/Rust/other bindings possible
5. **Cross-Platform** - .NET runs everywhere, native libs per platform
6. **NuGet Distribution** - Easy consumption via package manager

Would you like me to start implementing the C API layer for the VFS module?