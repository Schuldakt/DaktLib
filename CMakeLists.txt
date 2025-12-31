cmake_minimum_required(VERSION 4.1.1)
project(DaktLib 
    VERSION 1.0.0 
    LANGUAGES CXX
    DESCRIPTION "Modular C++ library for third-party game tooling"
)

# ============================================================================
# Global Settings
# ============================================================================

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Output directories
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# ============================================================================
# Build Options
# ============================================================================

option(DAKT_BUILD_CORE      "Build Core module"      ON)
option(DAKT_BUILD_CONFIG    "Build Config module"    ON)
option(DAKT_BUILD_LOGGER    "Build Logger module"    ON)
option(DAKT_BUILD_EVENTS    "Build Events module"    ON)
option(DAKT_BUILD_VFS       "Build VFS module"       ON)
option(DAKT_BUILD_PARSER    "Build Parser module"    ON)
option(DAKT_BUILD_GUI       "Build GUI module"       ON)
option(DAKT_BUILD_OVERLAY   "Build Overlay module"   ON)
option(DAKT_BUILD_OCR       "Build OCR module"       ON)
option(DAKT_BUILD_EXPORT    "Build Export module"    ON)
option(DAKT_BUILD_TESTS     "Build unit tests"       ON)
option(DAKT_BUILD_EXAMPLES  "Build examples"         OFF)
option(DAKT_BUILD_APPS      "Build applications"     ON)

# ============================================================================
# External Dependencies (FetchContent)
# ============================================================================

include(FetchContent)

# mbedtls - Lightweight crypto library for AES/Salsa20
FetchContent_Declare(
    mbedtls
    GIT_REPOSITORY https://github.com/Mbed-TLS/mbedtls.git
    GIT_TAG        v3.6.2
    GIT_SHALLOW    TRUE
)

# zlib - Compression for CryPak/P4K archives
FetchContent_Declare(
    zlib
    GIT_REPOSITORY https://github.com/madler/zlib.git
    GIT_TAG        v1.3.1
    GIT_SHALLOW    TRUE
)

# Configure mbedtls options before fetching
set(ENABLE_TESTING OFF CACHE BOOL "" FORCE)
set(ENABLE_PROGRAMS OFF CACHE BOOL "" FORCE)
set(MBEDTLS_FATAL_WARNINGS OFF CACHE BOOL "" FORCE)

# Fetch dependencies
message(STATUS "Fetching external dependencies...")
FetchContent_MakeAvailable(mbedtls zlib)

# ============================================================================
# Compiler Configuration
# ============================================================================

if(MSVC)
    add_compile_options(
        /W4             # Warning level 4
        /WX             # Treat warnings as errors
        /permissive-    # Strict conformance
        /Zc:__cplusplus # Correct __cplusplus macro
        /utf-8          # UTF-8 source and execution charset
    )
    add_compile_definitions(
        _CRT_SECURE_NO_WARNINGS
        NOMINMAX        # Disable min/max macros
        WIN32_LEAN_AND_MEAN
    )
else()
    add_compile_options(
        -Wall
        -Wextra
        -Wpedantic
        -Werror
        -Wno-unused-parameter
    )
endif()

# ============================================================================
# Modules
# ============================================================================

if(DAKT_BUILD_CORE)
    add_subdirectory(Core)
endif()

if(DAKT_BUILD_LOGGER)
    add_subdirectory(Logger)
endif()

if(DAKT_BUILD_CONFIG)
    add_subdirectory(Config)
endif()

if(DAKT_BUILD_EVENTS)
    add_subdirectory(Events)
endif()

if(DAKT_BUILD_VFS)
    add_subdirectory(VFS)
endif()

if(DAKT_BUILD_PARSER)
    add_subdirectory(Parser)
endif()

if(DAKT_BUILD_GUI)
    add_subdirectory(GUI)
endif()

if(DAKT_BUILD_OVERLAY)
    add_subdirectory(Overlay)
endif()

if(DAKT_BUILD_OCR)
    add_subdirectory(OCR)
endif()

if(DAKT_BUILD_EXPORT)
    add_subdirectory(Export)
endif()

# ============================================================================
# Applications
# ============================================================================

# ============================================================================
# Testing
# ============================================================================

if(DAKT_BUILD_TESTS)
    enable_testing()
    # add_subdirectory(Tests)
endif()

# ============================================================================
# Installation
# ============================================================================

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

install(EXPORT DaktLibTargets
    FILE DaktLibTargets.cmake
    NAMESPACE Dakt::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/DaktLib
)

configure_package_config_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake/DaktLibConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/DaktLibConfig.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/DaktLib
)

write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/DaktLibConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/DaktLibConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/DaktLibConfigVersion.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/DaktLib
)