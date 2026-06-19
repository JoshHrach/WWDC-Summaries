# Behind the Scenes of the Xcode Build Process
**WWDC18 · Session 415** · [Watch](https://developer.apple.com/videos/play/wwdc2018/415/)

_Platforms:_ iOS, macOS, watchOS, tvOS (Xcode 10 build system)

## Overview
This session demystifies what happens when you press Build in Xcode. It covers how Xcode orchestrates the build system to compile source files, link object files, and produce an application bundle — explaining the roles of the compiler front-ends (`clang`, `swiftc`), the linker (`ld`), and the supporting tools that handle assets, entitlements, code signing, and more.

The session explains how the build system discovers tasks (from build rules and build phases), how it models dependencies between tasks, and how it schedules work in parallel. It then dives into how `clang` parses and compiles Objective-C and Swift files, how the Swift compiler handles whole-module optimization and incremental compilation, and how the linker resolves symbol references to produce a Mach-O binary. Understanding these internals helps developers make informed decisions that directly affect build times and binary correctness.

## Key Topics

### The Build System
- Xcode 10 ships with a new build system (introduced in preview in Xcode 9) that replaces the legacy build system; it has a better dependency model and improved parallelism.
- Build tasks are derived from build phases (Compile Sources, Copy Bundle Resources, Link Binary with Libraries, Run Script) and build rules (per-file-type processing rules).
- The build system constructs a dependency graph of tasks; tasks with no dependency relationships run in parallel on all available CPU cores.
- Declared inputs and outputs of Run Script build phases are used by the build system to determine whether a script needs to re-run and to schedule it correctly relative to other tasks.
- `xcodebuild` is the command-line interface to the same build system used by the Xcode IDE.

### Compiling with clang (Objective-C / C / C++)
- Preprocessing: the preprocessor expands `#import` / `#include` directives and macros, producing a single translation unit.
- Parsing and semantic analysis: clang constructs an Abstract Syntax Tree (AST) and performs type-checking.
- Code generation: the AST is lowered to LLVM IR, optimized, and compiled to machine code producing an object file (`.o`).
- Each source file in the Compile Sources phase is compiled independently, enabling fine-grained parallelism and incremental compilation (only changed files are recompiled).
- Precompiled headers (PCH) and Clang modules (`.modulemap`) amortize repeated header parsing across compilation units.
- `-index-store-path` writes an index used by Xcode's editor features (jump to definition, callers, etc.).

### Compiling with swiftc (Swift)
- Swift compilation is more complex because the type checker needs cross-file information within a module.
- **Whole-Module Optimization (WMO)**: compiles all Swift files in a target together as a single unit; enables more aggressive optimization and inlining but produces a single compilation task (no per-file parallelism). Used for Release builds.
- **Incremental compilation**: compiles files individually using a dependency graph (`*.swiftdeps` files) to determine which files must be recompiled when a declaration changes. Used for Debug builds in Xcode 10.
- `swiftc` produces object files, a `.swiftmodule` (serialized type info for the module), and a generated Objective-C header (`-Swift.h`) for cross-language interoperability.
- Swift's `-Onone` (Debug), `-O` (Release), and `-Osize` (size-optimized) optimization levels.

### Linking
- The linker (`ld`) combines object files and static libraries into a Mach-O executable or dylib.
- Linking resolves symbol references: each object file exports symbols and imports symbols; the linker matches them up.
- Dynamic linking: at link time, the linker records which dylibs are needed (`@rpath`, `@executable_path`, `@loader_path` references); at launch time, `dyld` loads and binds them.
- Weak linking: symbols marked `__attribute__((weak_import))` (or Swift's `@available`) are optional; the binary can run on older OS versions where the symbol is absent, tested with `nil` or version checks.
- Two-level namespace: each imported symbol includes the library it came from, preventing name collisions and speeding up symbol lookup.
- Dead-code stripping (`-dead_strip`): the linker removes unreachable code and data, reducing binary size.

### Code Signing and Packaging
- After linking, `codesign` signs the binary and bundle using the provisioning profile and signing certificate.
- `actool` compiles asset catalogs (`.xcassets`) into `Assets.car`.
- `ibtool` / `IBCompiler` compiles Interface Builder files (`.xib`, `.storyboard`) into `.nib` format.
- `dsymutil` extracts DWARF debug info from object files and the linked binary into a `.dSYM` bundle for crash symbolication.

### Build Phases and Run Script Phases
- Run Script phases execute arbitrary shell scripts; they should declare inputs and outputs so the build system can correctly schedule them and skip them when inputs haven't changed.
- Undeclared dependencies between a script and other tasks can cause ordering bugs and non-deterministic builds.
- Scripts that always re-run even when not needed can significantly slow incremental builds.

## APIs & Frameworks

**Xcode / Build System Tools**
- `xcodebuild` — command-line build tool
- Xcode new build system (default in Xcode 10) — improved dependency graph and parallelism
- Build phases: Compile Sources, Copy Bundle Resources, Link Binary with Libraries, Run Script
- Build rules — per-file-type processing rules; define inputs/outputs
- `clang` — C/Objective-C/C++ front-end and driver
- `swiftc` — Swift compiler driver
- `ld` — Apple linker (produces Mach-O)
- `dyld` — dynamic linker / loader
- `actool` — asset catalog compiler
- `ibtool` / IBCompiler — Interface Builder file compiler
- `dsymutil` — DWARF/dSYM bundle extractor
- `codesign` — binary and bundle signing tool
- Precompiled headers (`.pch`) — amortize header parsing
- Clang modules / module maps (`.modulemap`) — modular import system
- `*.swiftdeps` files — Swift incremental compilation dependency graph
- `.swiftmodule` — serialized Swift module interface
- `-Swift.h` generated Objective-C header — Swift-to-Objective-C interop
- LLVM IR — intermediate representation between front-end and machine code
- Mach-O binary format — executable, dylib, framework binary format
- `@rpath`, `@executable_path`, `@loader_path` — dynamic library search path tokens
- Dead-code stripping (`-dead_strip` linker flag)
- Whole-Module Optimization (`-whole-module-optimization` / `-wmo`)
- `-Onone`, `-O`, `-Osize` — Swift optimization levels

## Code Highlights

Declaring inputs and outputs in a Run Script build phase (shell):
```sh
# In the "Input Files" section of the Run Script phase:
$(SRCROOT)/Scripts/generate_version.sh

# In the "Output Files" section:
$(DERIVED_FILE_DIR)/GeneratedVersion.swift
```

Checking the effective compilation mode (e.g., in a build script):
```sh
if [ "$SWIFT_COMPILATION_MODE" = "wholemodule" ]; then
    echo "WMO enabled"
fi
```

Weak-linking a symbol for backward deployment (Objective-C):
```objc
// Link against the framework as "Optional" (weak link) in the target's
// Build Phases > Link Binary with Libraries.
// At runtime:
if (NSClassFromString(@"SomeNewClass") != nil) {
    // use SomeNewClass
}
```

## Takeaways
- The build system runs tasks in parallel based on a dependency graph; correctly declaring Run Script inputs and outputs is essential for correctness and performance.
- Swift's incremental compilation in Debug and WMO in Release are fundamentally different strategies — understanding the trade-offs helps you choose the right settings.
- The linker's dead-code stripping, two-level namespace, and weak-linking mechanisms are key tools for smaller, safer, backward-compatible binaries.
- Tools like `dsymutil` and `codesign` are automatic parts of the build pipeline; knowing they exist helps when diagnosing crash symbolication or signing failures.

---
_Source: WWDC18 Session 415 page (abstract and resource links); transcript unavailable on page._
