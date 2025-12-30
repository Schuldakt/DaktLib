# Copilot Instructions for DaktLib

## Overview

This document defines how Copilot should operate within the **DaktLib** repository.  
DaktLib contains multiple reusable modules for Star Citizen third-party tooling written in **C++23** and built with **CMake (x64)**.  
Each module exists as a submodule with its own `ARCHITECTURE.md` and `TODO.md`.

---

## Core Principles

Copilot **must**:

1. Focus on **one module at a time**.
2. Follow the module’s `TODO.md` **in strict order**.
3. Adhere exactly to each module’s `ARCHITECTURE.md`.
4. Update both the **module** and **root** `TODO.md` files after each completed task.
5. Continue **automatically** to the next TODO item until all are complete.
6. Never deviate from architecture constraints.

---

## Workflow

### 1. Module Context Initialization

Before generating code:

- Load `module/ARCHITECTURE.md`.
- Load `module/TODO.md`.

Stop and raise an issue if any file cannot be parsed or is missing.

---

### 2. TODO Processing

For each item in the module’s `TODO.md`:

1. Interpret the task.
2. Implement the required changes according to architecture.
3. Write necessary code and tests.
4. Update source files only as required by the task.
5. Confirm that:
   - Code compiles successfully with CMake (x64, C++23).
   - Tests pass if applicable.
6. Update:
   - The **module TODO.md** — mark as complete with a one-line summary.
   - The **root TODO.md** — indicate completion under the corresponding module.
7. Commit using the **standard commit message format**.

Do not skip, batch, or reorder TODO items.

---

### 3. Architecture Compliance

Copilot must respect:

- Directory structures
- Naming conventions
- Public/private API organization
- Dependency rules

No architectural deviation is permitted unless explicitly documented.

If a TODO cannot be implemented due to architectural conflict:

- Create an issue in the module’s issue tracker.
- Do not proceed until the conflict is resolved.

---

### 4. Continuous Progression

After completing one TODO item:

- Load the next item automatically.
- Proceed until the module’s TODO list is finished.
- When finished:
  - Mark module complete in the root TODO.
  - Move to the next unfinished module.

Copilot must proceed without waiting for manual prompts.

---

### 5. CMake and Build Rules

All changes must preserve a working build:

- Respect existing `CMakeLists.txt` structure.
- Add new targets or dependencies only if required and permitted by architecture.
- Ensure the project builds for **x64** with **C++23**.

---

## Commit Message Format

[module/<module-name>] TODO <ID>: <Short summary>
* Completed according to TODO description.
* Conforms to ARCHITECTURE.md.
* Unit tests updated/added if applicable.
* Updated module and root TODO.md.


---

## Testing

- Add or update unit tests wherever functionality is introduced or modified.
- Use existing test frameworks and conventions.
- Ensure all tests pass before updating TODOs.

---

## Issue Handling

When ambiguity or conflict arises:

1. Create a concise issue in the module’s issue tracker.
2. Reference the conflicting TODO item.
3. Do not proceed with implementation until clarified.

---

## Completion Criteria

A module is considered complete when:

- All TODO items are marked complete.
- Corresponding entries in the root TODO are marked complete.
- The module builds successfully.
- All tests pass.

---
