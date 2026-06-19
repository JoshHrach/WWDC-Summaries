# Demystify Parallelization in Xcode Builds
**WWDC22 · Session 110364** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110364/)

_Platforms:_ macOS (Xcode 14, Swift 5.7)

## Overview
This session explains how the Xcode build system extracts maximum parallelism from multi-target Swift projects. It covers the fundamentals of the build task graph, introduces two major Xcode 14 optimizations — **eager module emission** and **eager linking** — and introduces the **Build Timeline**, a new visual tool for diagnosing build performance bottlenecks.

The session is split between Xcode Build System concepts (task graph, build phases, script phase parallelization and sandboxing) and Swift Compiler-level optimizations that reduce idle CPU time in large projects by unblocking downstream targets sooner.

## Key Topics

### Build Task Graph Fundamentals
Every build is a directed acyclic graph (DAG) of tasks. Each task has inputs and outputs; a task cannot start until all tasks producing its inputs have completed. The **critical path** — the longest dependency chain — defines the theoretical minimum build time regardless of available CPU cores. Shortening the critical path is the primary strategy for improving build scalability.

The **Build Timeline** (new in Xcode 14) visualizes this graph after a build completes. It replaces the hierarchy-based build log view for performance analysis:
- Row count at any time = degree of parallelism
- Element width = task duration
- Empty space = tasks blocked waiting on upstream outputs
- Colors = target associations
- Available in Xcode via Editor → Assistant from the Build Log

### Build Phase Parallelization Within a Target
Xcode automatically runs build phases in parallel when their inputs/outputs are independent. For example, compilation and resource copying run in parallel because neither depends on the other's outputs; linking always follows compilation because it consumes object files.

**Run Script phases** are serialized by default because their inputs/outputs must be manually declared. Two settings enable parallelization:
- `FUSE_BUILD_SCRIPT_PHASES = YES` — allows the build system to run script phases in parallel when their declared inputs/outputs allow it
- `ENABLE_USER_SCRIPT_SANDBOXING = YES` — sandbox blocks script phases from reading or writing undeclared files; violations cause an immediate build failure with a list of unauthorized file paths

Used together, sandboxing guarantees correct dependency declarations, which enables `FUSE_BUILD_SCRIPT_PHASES` to safely parallelize script phases and also enables more accurate incremental build skipping.

### Swift Compiler Integration in Xcode 14
In Xcode 14, the Swift Driver is reimplemented in Swift and **fully integrated** into the Xcode Build System. All Swift compilation sub-tasks (individual file compilations, batch jobs, module merging, linking) are now in the central build system scheduler rather than in isolated per-target Driver processes. This allows the build system to make fine-grained scheduling decisions across all targets simultaneously, reducing idle CPU cores.

**Incremental compilation mode** (Debug builds): the Driver breaks each target's source files into parallel compile tasks, potentially batching files together. The build log shows individual file diagnostics per batch job. Use `Swift Compiler - Code Generation: Compilation Mode = Incremental` in Debug schemes to enable this.

### Eager Module Emission (New in Xcode 14 / Swift 5.7)
Previously, a downstream target could not begin compilation until the upstream target had **finished all compilation and produced a merged `.swiftmodule`**. In Xcode 14, each Swift target emits its module in a **separate `emit-module` task** that runs directly from source in parallel with the compilation tasks. Downstream targets unblock as soon as `emit-module` completes — without waiting for all compile tasks to finish — dramatically reducing idle time in wide dependency graphs.

### Eager Linking (New in Xcode 14 / Swift 5.7)
Without eager linking, a target's link task cannot start until all its dependencies have produced their linked products. With eager linking, a dependent target's link step depends on a **text-based dynamic library stub** (`.tbd`) produced during `emit-module` rather than the full linked product. This lets the dependent target begin linking in parallel with its dependency's link phase, shortening the critical path.

Enable with the Xcode build setting: `EAGER_LINKING = YES`.

Applies to: pure Swift targets that are dynamically linked by their dependents.

## APIs & Frameworks

### Xcode Build System
- **Build Timeline** **[NEW in Xcode 14]** — visual parallelization view of the build log; open via Editor → Assistant in Build Log
- Incremental build: tasks are skipped when inputs are unchanged and outputs are up-to-date
- Build phases run in parallel when inputs/outputs are independent

### Xcode Build Settings
- `FUSE_BUILD_SCRIPT_PHASES = YES` — enables parallel execution of Run Script phases when dependency information is correct
- `ENABLE_USER_SCRIPT_SANDBOXING = YES` — sandboxes shell scripts; blocks access to undeclared inputs/outputs within the source root and derived data directory
- `EAGER_LINKING = YES` **[NEW in Xcode 14]** — enables early linking of dependent targets via `.tbd` stubs
- `Swift Compiler - Code Generation: Compilation Mode = Incremental` — enables per-file parallel compilation in Debug builds

### Swift Driver (Open Source, Xcode 14)
- Fully rewritten in Swift; integrated into Xcode Build System as first-class tasks
- **Emit-module task** **[NEW in Swift 5.7]** — produces a target's `.swiftmodule` as a separate parallel task, unblocking downstream targets sooner
- **Batch compilation** — groups source files into parallel batch jobs based on build system heuristics
- Repository: [github.com/apple/swift-driver](https://github.com/apple/swift-driver)

## Code Highlights

No code samples in this session. Key configuration examples:

Enable user script sandboxing in an xcconfig or build settings editor:
```
ENABLE_USER_SCRIPT_SANDBOXING = YES
```

Enable parallel script phases (requires correct input/output declarations):
```
FUSE_BUILD_SCRIPT_PHASES = YES
```

Enable eager linking for Swift dynamic framework targets:
```
EAGER_LINKING = YES
```

Example of a two-phase script dependency that requires sandboxing to be detected correctly:
- Phase 1 (`Calculate Checksum`): reads `input.txt`, writes `checksum.txt` to `DERIVED_FILE_DIR`
- Phase 2 (`Generate HTML`): reads `input.txt` and `checksum.txt`, writes `output.html`
- Without sandboxing and with missing input declaration on Phase 2, these phases can run concurrently, causing Phase 2 to read a stale or missing `checksum.txt`. Sandboxing immediately fails the build and reports the undeclared file access.

## Takeaways
- The Build Timeline (Xcode 14) is the primary tool for diagnosing build bottlenecks; look for empty space (blocked tasks) and wide tasks (long-running bottlenecks) on the critical path.
- Xcode 14 integrates the Swift Driver fully into the build scheduler, enabling cross-target sub-task scheduling and eliminating per-target isolation that caused idle CPU cores.
- The new `emit-module` task (Swift 5.7) unblocks downstream target compilation much earlier by producing the module interface independently of per-file compilation tasks.
- Enable `EAGER_LINKING` to let downstream targets begin linking in parallel with upstream link tasks using `.tbd` stubs.
- Use `ENABLE_USER_SCRIPT_SANDBOXING` to surface missing input/output declarations in Run Script phases, then set `FUSE_BUILD_SCRIPT_PHASES` to run correctly declared script phases in parallel.

---
_Source: WWDC22 Session 110364 page (abstract, transcript, and resource links)._
