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

## Current Status Snapshot

| Module | Status | Notes |
|--------|--------|-------|
| Core | Scaffolded | Interfaces/types in place; default impls optional. |
| Logger | Scaffolded/Basic | Console/file/async sinks and C API sketched; formatter to expand. |
| Events | Scaffolded | API and C API stubs; dispatch logic pending. |
| VFS | Scaffolded | Backends/interfaces stubbed; real I/O not wired. |
| Parser | Scaffolded | Text + game/asset format parsers stubbed. |
| SQL | Scaffolded | SQLite vendor slot; wrappers and builders stubbed. |
| Export | Scaffolded | FBX exporter (stage/layer/prim, PBR, animation) planned; writer not implemented. |
| Decrypt | Scaffolded | CryEngine chunk detection/crypto/comp/hash registries stubbed. |
| Http | Scaffolded | Platform backends and protocol logic stubbed. |
| Capture | Scaffolded | Platform capture backends stubbed. |
| ML | Scaffolded | Runtime-loaded backends (ORT/DML/TensorRT) stubbed. |
| OCR | Scaffolded | Backends (Windows.Media.Ocr/ONNX) stubbed; preprocessing pending. |
| Overlay | Scaffolded | Surfaces/platform wiring stubbed; painter/batching pending. |
| GUI | Scaffolded | Immediate/retained API, layout, text, backends stubbed. |

## Near-Term Priorities
- Implement FBX writer core: object/ID registry, property encoding, connections, chunked buffers; basic geometry/material/animation emit.
- Bring Logger/Events to usable state: formatter tokens, file rotation, thread-safe dispatch, C API round-trips.
- Stand up Http request/response parsing and one backend (WinHTTP or POSIX) with TLS abstraction.
- Implement one capture backend (DXGI or WGC) end-to-end with frame surfaces and format conversion.
- Wire Overlay surfaces and window creation (WS_EX_LAYERED + DirectComposition first), plus painter batching/clip stack.
- Parser: finish JSON/XML SAX + DOM basics and binary reader/writer; add minimal P4K/CryModel chunk readers.
- SQL: drop in sqlite3 amalgamation and implement Connection/Statement/ResultSet MVP.
- ML: dynamic loader for ONNX Runtime, Tensor basics, session run stub; OCR: Windows.Media.Ocr backend and preprocessing pipeline.
- VFS: physical + memory backend, mount resolution, and core stream helpers.

## Milestones (updated)

| Milestone | Target | Scope |
|-----------|--------|-------|
| **v0.1.0** | Core+Logger+Events usable | Interfaces stable; basic sinks/dispatch/formatter; CMake/tooling ready. |
| **v0.2.0** | Data scaffolding solid | VFS physical/memory, Parser JSON/XML basics, SQL MVP with sqlite3 vendor, Decrypt detection pass. |
| **v0.3.0** | Export & Http/Capture MVP | FBX writer minimal geometry/material emit; Http single backend; Capture single backend. |
| **v0.4.0** | Vision/Compute | ML loader + run stub, OCR Windows backend, Parser game formats pass. |
| **v0.5.0** | Presentation | Overlay windowing + painter basics; GUI immediate core/drawlist/text shaping stub across one backend. |
| **v1.0.0** | Cross-platform polish | Multi-backend coverage, C API hardening, bindings/packaging. |

## Estimated Effort (high level)
- Core/Logger/Events hardening: ~3-4 weeks
- Data layer MVP (VFS/Parser/SQL/Decrypt): ~10-14 weeks
- Export (FBX) MVP: ~8-10 weeks
- Http/Capture MVP: ~8-10 weeks
- ML/OCR MVP: ~8-10 weeks
- Overlay/GUI MVP: ~12-16 weeks
- Packaging/bindings/polish: ~6-8 weeks (overlapping)

## Risk Register

| Risk | Impact | Mitigation |
|------|--------|------------|
| EAC false positive | High | Stay OS-API-only; review capture/overlay/OCR paths. |
| FBX interop gaps | Medium | Test against major DCCs; keep ASCII debug path; add golden files. |
| Platform API drift (WinHTTP, DXGI, WS_EX_LAYERED, PipeWire, CFNetwork) | Medium | Isolate backends; guard with feature checks. |
| Binding/ABI churn | Medium | Freeze C APIs before 1.0; use versioned symbols where needed. |
| Scope creep (GUI/Parser breadth) | High | Enforce MVPs per milestone; defer optional formats/widgets. |

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
