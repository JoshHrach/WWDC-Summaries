# Debug Swift Debugging with LLDB
**WWDC22 · Session 110370** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110370/)

_Platforms:_ macOS, Xcode 14

## Overview
This session takes a deep dive into how LLDB works internally and what information it needs from the build system to function correctly. Using a Swift text-adventure game that links against a third-party framework built on a CI server, the session walks through diagnosing two classes of problems: missing source code (a debug info / path remapping issue) and a broken expression evaluator (a missing Swift module issue).

The key insight is that LLDB has a dual nature: it is both a **debugger** (using debug info and reflection metadata for the variable view and `v`/`frame variable` commands) and an embedded **compiler** (using module imports for the expression evaluator, `p`, and `po`). Understanding this separation is essential for debugging complex multi-machine or multi-target Swift projects.

A new `swift-healthcheck` command in LLDB (Xcode 14) surfaces diagnostics from the embedded Swift compiler, making it the recommended first stop when `po` or `p` stops working.

## Key Topics

### Debug Info and Source Path Remapping
When a compiler builds a function it embeds source file paths into debug info as "breadcrumbs" so the debugger can map machine addresses back to source lines. On Apple platforms, debug info lives in object files and is linked into `.dSYM` bundles by `dsymutil`. LLDB uses Spotlight to locate `.dSYM` bundles automatically.

When code is built on a CI server, embedded source paths point to the server's file system, not the developer's local machine. Two remediation strategies:
- **LLDB source map** (`settings set target.source-map`): remap a path prefix at debug time; put the command in a per-project `.lldbinit` file via the Scheme editor
- **Compiler `-debug-prefix-map`**: replace machine-specific path prefixes with a canonical placeholder at build time so a single LLDB remap covers all build machines

Source path remapping works for Swift, C++, and Objective-C alike.

### LLDB's Dual Nature: Debugger vs. Compiler
| Capability | Commands | Data Source |
|---|---|---|
| Debugger | `v`, `frame variable` | Debug info + Swift reflection metadata |
| Compiler | `expr`, `p`, `po` | Modules (Swift/Clang) |

This separation is new and explicit in Xcode 14. The variable view and `v` command can work even when `po` fails, because they rely only on debug info rather than module imports.

### swift-healthcheck (New in Xcode 14)
`swift-healthcheck` prints the configuration log of LLDB's embedded Swift compiler, including which modules were attempted and which failed to import. Run it immediately after a failed `p`/`po` to identify missing modules.

### How LLDB's Compiler Finds Swift Modules
LLDB's embedded Swift compiler searches for modules from many sources in priority order:
- System framework `.swiftinterface` files in the SDK
- Binary `.swiftmodule` files embedded in `.dSYM` bundles (for dynamic libraries and executables)
- Clang modules (header files with module maps)
- Bridging headers
- Debug-info-reconstructed type information (fallback)

### Swift Modules in Static Archives
Static archives are not produced by the linker — they are collections of object files. As a result, Swift modules belonging to static archives are **not** automatically registered with the linker and will not be picked up by `dsymutil`. Every dynamic library or executable that links a static archive must explicitly register its Swift module using the linker flag `-add_ast_path`. Verify registration with `dsymutil -s MyApp | grep .swiftmodule`.

On non-Apple platforms (Linux), use `swiftc -modulewrap My.swiftmodule -o My.swiftmodule.o` to embed the module in an object file that can be linked normally.

### Serialized Search Paths in Swift Modules
The Swift compiler serializes Clang header search paths and related options into binary `.swiftmodule` files. This is useful during local builds but causes failures when the module is shipped to a machine with a different file system layout. Use `-no-serialize-debugging-options` (Xcode setting: `SWIFT_SERIALIZE_DEBUGGING_OPTIONS=NO`) before distributing binary modules. Search paths can be re-introduced in LLDB at debug time with `target.swift-extra-clang-flags`, `target.swift-framework-search-paths`, and `target.swift-module-search-paths`.

## APIs & Frameworks

### LLDB Commands
- `image list` — lists all loaded dylibs and their associated `.dSYM` bundles
- `image lookup -va $pc` — shows full debug info (including embedded source path) for the current instruction pointer
- `settings set target.source-map <prefix> <new>` — remaps a source path prefix at debug time
- `settings list target.source-map` — shows current source map entries
- `v words` / `frame variable words` — display a variable using debug info (no compiler required)
- `p words` / `expr words` — evaluate an expression using the embedded Swift compiler
- `po words` / `expr -O -- words` — print the object description using the embedded Swift compiler
- `s` / `thread step-in` — step into a function call
- `n` / `thread step-over` — step over an instruction
- `swift-healthcheck` **[NEW]** — prints the embedded Swift compiler's configuration log; shows failed module imports
- `mem read UnsafePointer<Items>(self.inventory)` — raw memory read of a Swift variable

### Compiler Flags
- `-debug-prefix-map $PWD=/BUILDROOT` **[NEW recommended pattern]** — canonicalizes source paths in debug info at build time
- `-no-serialize-debugging-options` — prevents embedding local search paths in binary `.swiftmodule` files
- `SWIFT_SERIALIZE_DEBUGGING_OPTIONS=NO` — Xcode build setting equivalent

### Linker Flags
- `ld … -add_ast_path /path/to/My.swiftmodule` — registers a Swift module with the linker so `dsymutil` can collect it into the `.dSYM` bundle

### Swift Driver (Linux/non-Apple)
- `swiftc -modulewrap My.swiftmodule -o My.swiftmodule.o` — wraps a Swift module in an object file for linking

### LLDB Settings for Search Path Override
- `settings set target.swift-extra-clang-flags …`
- `settings set target.swift-framework-search-paths …`
- `settings set target.swift-module-search-paths …`

## Code Highlights

Listing loaded dylibs and verifying `.dSYM` association:
```
image list
```

Inspecting debug info at the current instruction pointer (reveals server-side source path):
```
image lookup -va $pc
```

Remapping a build-server source path prefix in LLDB (put in per-project `.lldbinit`):
```
settings set target.source-map /Volumes/BUILD_SERVER/projects /Users/demo/Desktop/Adventure/3rdparty
```

Canonicalizing source paths at compile time so one remap covers all machines:
```
-debug-prefix-map $PWD=/BUILDROOT
```

Diagnosing a failed `po` with the new healthcheck command:
```
swift-healthcheck
```

Verifying that a static archive's Swift module was registered with the linker:
```
dsymutil -s MyApp | grep .swiftmodule
```

Registering a Swift module from a static archive with the Apple linker:
```
ld … -add_ast_path /path/to/My.swiftmodule
```

Wrapping a Swift module for linking on Linux:
```bash
swiftc -modulewrap My.swiftmodule -o My.swiftmodule.o
```

## Takeaways
- LLDB is both a debugger (debug info + reflection metadata → `v`/`frame variable`) and a compiler (module imports → `expr`/`p`/`po`). The two sides fail independently; always check `swift-healthcheck` when `po` breaks.
- Source paths embedded in debug info point to the machine where the code was compiled. Use `settings set target.source-map` (or a project `.lldbinit` file) to remap them; use `-debug-prefix-map` at the compiler to canonicalize them before distribution.
- Static archive Swift modules must be explicitly registered with the linker via `-add_ast_path`; otherwise `dsymutil` cannot collect them into the `.dSYM` bundle and LLDB's expression evaluator cannot find the types.
- Before shipping a binary `.swiftmodule` to another machine, build with `-no-serialize-debugging-options` / `SWIFT_SERIALIZE_DEBUGGING_OPTIONS=NO` to avoid broken local search paths in the distributed module.

---
_Source: WWDC22 Session 110370 page (abstract, transcript, code samples, and resource links)._
