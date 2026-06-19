# Demystify Explicitly Built Modules
**WWDC24 · Session 10171** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10171/)

_Platforms:_ Xcode 16, macOS 15

## Overview
Xcode 16 changes how Swift and Clang modules are compiled, introducing a new build mode called Explicitly Built Modules (EBM). In prior Xcode versions, modules were built implicitly as a side effect of compiling source files, leading to redundant work and unpredictable build parallelism. EBM makes module compilation an explicit, schedulable, cacheable build step — improving build times, enabling better dependency visualization, and fixing a class of build reliability issues caused by implicit module races.

This session explains the difference between implicit and explicit module builds, how Xcode 16 adopts EBM automatically for new and existing projects, where developers may encounter issues during adoption, and how to diagnose those issues using new Xcode build log features.

## Key Topics
- **Implicit vs. explicit module builds** — in implicit mode the compiler discovers and builds modules on-demand during compilation; in explicit mode, Xcode pre-scans all targets, determines module dependencies, builds modules as discrete tasks, and passes pre-built module paths to each compiler invocation.
- **Automatic adoption in Xcode 16** — most projects benefit automatically; the build system detects which targets support EBM and opts them in without any project changes.
- **Build log improvements** — a new "Module Dependencies" section in the build log groups module compilation tasks separately, making it easy to see which modules are rebuilt and why.
- **Common adoption issues** — header files that produce different module content depending on preprocessor context (e.g., `#if SOME_DEFINE`) can cause module hash mismatches; fix by making headers self-contained or using umbrella header patterns.
- **Module cache and build cache integration** — EBM modules participate in Xcode's build cache and Xcode Cloud's distributed build cache, reducing redundant module rebuilds across machines.

## APIs & Frameworks

**Xcode 16 Build System**
- **[NEW]** Explicitly Built Modules (EBM) build mode — automatic for most targets in Xcode 16; replaces implicit module builds
- **[NEW]** Module dependency scanning phase — a new pre-build scan step that maps all module imports and produces an explicit dependency graph
- **[NEW]** "Module Dependencies" section in Xcode build log — lists each module compiled, its dependencies, and whether it was a cache hit
- **ENABLE_MODULE_VERIFIER** build setting — existing; unchanged; independent of EBM
- **SWIFT_ENABLE_EXPLICIT_MODULES** build setting — **[NEW]** opt-in/opt-out flag for Swift targets (default: automatic)
- **CLANG_ENABLE_EXPLICIT_MODULES** build setting — **[NEW]** opt-in/opt-out flag for Clang targets (default: automatic)
- Xcode Build Cache — existing infrastructure; EBM tasks participate for cache hits across clean builds
- Xcode Cloud distributed cache — EBM module artifacts are shared across CI runs

**Swift / Clang Compiler (underlying)**
- `-explicit-swift-module-map-file` Swift flag — compiler flag Xcode generates and passes automatically; not for manual use
- `-fmodule-file=` Clang flag — analogous Clang flag; similarly generated automatically
- Clang module map files (`.modulemap`) — unchanged authoring; EBM consumes them the same way

## Code Highlights
There is no new source-code API — EBM is a build-system-level change. The relevant developer action is diagnosing build failures:

1. Open the Xcode build log and expand the "Module Dependencies" section.
2. Look for modules that are rebuilt on every build (no cache hit) — these usually have context-dependent headers.
3. Inspect problem headers: ensure `#include` guards are correct, and that no macros defined by the including source file affect the module's content.

To temporarily disable EBM for a target while diagnosing:

```
// In Xcode: target Build Settings → search "Explicit Modules"
SWIFT_ENABLE_EXPLICIT_MODULES = NO
CLANG_ENABLE_EXPLICIT_MODULES = NO
```

## Takeaways
- EBM is on by default in Xcode 16 — most projects will see faster incremental and clean builds with no changes; the improvement is largest for projects with many shared modules.
- If a target's build breaks after upgrading to Xcode 16, check the Module Dependencies build log section first; the most common cause is a header that emits different content depending on which source file includes it.
- EBM enables the build system to parallelize module compilation independently of source compilation — the resulting build graph is wider and completes earlier on multi-core machines.
- Module artifacts built under EBM are cached by Xcode Cloud, so CI clean builds become dramatically faster once the cache is warm.

---
_Source: WWDC24 Session 10171 page (abstract, chapter summaries, code samples, and resource links)._
