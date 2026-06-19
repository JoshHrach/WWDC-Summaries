# Meet Mergeable Libraries
**WWDC23 · Session 10268** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10268/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10

## Overview
Mergeable libraries are a new linking model introduced in Xcode 15 that combine the build-time benefits of dynamic libraries with the runtime performance of static libraries. Powered by a newly reimplemented static linker, mergeable libraries allow frameworks to be built with embedded metadata that the linker can later use to merge their contents directly into an app binary or a group framework.

In debug builds, the linker instead reexports the libraries (no merging occurs), preserving the fast incremental build times developers expect during development. In release builds, the libraries are fully merged and their binaries removed from disk, reducing bundle size and app launch time by lowering the number of frameworks dyld must load.

The feature is opt-in and configured entirely through Xcode build settings, with no source code changes required. Automatic mode merges all direct embedded framework dependencies into the app binary in one step; manual mode gives fine-grained control for complex dependency graphs.

## Key Topics

### Static vs. Dynamic Libraries Trade-offs
- Static libraries: fast launch, slow incremental builds (code copied at link time)
- Dynamic libraries: fast incremental builds, slower launch (dyld loads many frameworks at runtime)
- Mergeable libraries: offer both fast builds (debug = reexport) and fast launch (release = merged)

### How Mergeable Libraries Work
- A dynamic library is built with `-make_mergeable`, which embeds metadata into the binary
- The merged binary target uses `-merge_library` or `-merge_framework` linker flags
- Metadata enables the linker to treat the dependency like a static library at link time
- After merging, the individual mergeable library binaries are removed from disk
- The merged binary type (executable or framework) remains unchanged
- The linker deduplicates strings, Objective-C selectors, and `objc_msgSend` stubs across all merged libraries

### Automatic Merging (Xcode 15)
- Set `MERGED_BINARY_TYPE = automatic` on the app target
- All direct embedded framework dependencies (not system libraries) become mergeable and are merged into the app binary
- Recommended to also add `-Wl,-no_exported_symbols` to Other Linker Flags for apps that don't export symbols (reduces size/build time)
- Use `Exported Symbols File` build setting if app extensions require specific exported symbols

### Manual Merging (Xcode 15)
- Set `MERGED_BINARY_TYPE = manual` on the group framework target that will act as the merged output
- Set `MERGEABLE_LIBRARY = YES` on each framework that should be merged
- Libraries not marked `MERGEABLE_LIBRARY` remain on disk (useful for test support frameworks, app extensions, etc.)
- A "group library" framework is created to encapsulate the merged dependencies

### Debug Mode Behavior
- In debug builds, the linker reexports rather than merges, preserving dynamic linking performance
- Reexporting: implementation lives in one dylib but appears to come from the merged target
- All mergeable libraries stay on disk in debug mode
- Symbolication still works; stack traces show the merged binary path

### Considerations
- **Dependents**: any target that previously linked a mergeable library must be updated to link the merged binary instead (the mergeable library is removed from disk)
- **Autolinking**: compiler automatically passes framework link flags from module imports; update "Link Binary with Libraries" to the merged framework to avoid dynamic link failures
- **`dlopen` / `NSBundle`**: paths must point to the merged framework; `Bundle.module` / `NSBundle(for:)` require iOS 12+ deployment target due to a runtime hook added then; use `-no_merged_libraries_hook` linker option if bundle resources are not needed (improves launch time)
- **New linker only**: new linker options only work with Xcode 15's reimplemented linker; does not support `armv7k` (watchOS 8 and earlier); set deployment target to watchOS 9+ to use the new linker
- **XCFramework distribution**: mergeable metadata roughly doubles the dylib size; metadata is stripped from embedded copies, so app bundle size is not impacted

### Recommendations
- Link dependents against the merged binary, not individual mergeable libraries
- Set merging build settings at the Xcode target level to avoid unintentional side effects
- Consider converting static libraries to dynamic so they can become mergeable
- Use automatic mode for simple topologies; manual mode for complex dependency graphs with test targets or app extensions

## APIs & Frameworks

- **Mergeable Libraries** **[NEW]** – new linking model in Xcode 15 / static linker
- `MERGED_BINARY_TYPE` build setting **[NEW]** – values: `automatic`, `manual`
- `MERGEABLE_LIBRARY` build setting **[NEW]** – `YES` / `NO` on individual framework targets
- `-make_mergeable` linker flag **[NEW]** – builds a dynamic library with mergeable metadata
- `-merge_library` / `-merge_framework` linker flags **[NEW]** – merges dependencies into the output binary
- `-no_exported_symbols` linker flag – strips exports from app binaries that don't need them
- `-no_merged_libraries_hook` linker flag **[NEW]** – disables bundle lookup hook for merged frameworks
- `Exported Symbols File` build setting – controls exported symbol list for app extensions
- **`dyld`** (dynamic linker) – loads fewer frameworks at launch when libraries are merged
- **`dlopen`** – runtime dynamic loading; paths must target the merged framework
- `Bundle.module` (Swift) / `NSBundle(for:)` (Objective-C) – bundle resource APIs; require iOS 12+ with mergeable libraries
- **XCFramework** – distribution mechanism for mergeable libraries; Xcode and Swift Package Manager both supported

## Code Highlights

Enabling automatic merging (Xcode build setting):
```
MERGED_BINARY_TYPE = automatic
```

Disabling symbol exports for app binaries (Other Linker Flags):
```
-Wl,-no_exported_symbols
```

Enabling a framework as manually mergeable (Xcode build setting):
```
MERGEABLE_LIBRARY = YES
```

## Takeaways
- Mergeable libraries eliminate the traditional static-vs.-dynamic trade-off: debug builds use reexporting for fast iteration, release builds use merging for fast launch and smaller bundles.
- Adoption requires only Xcode build setting changes — no source code modifications.
- Any target previously linking a mergeable library must be updated to link the merged binary instead, since the individual library binaries are removed from disk.
- The minimum deployment target for `Bundle.module` / `NSBundle(for:)` resource lookup with mergeable libraries is iOS 12; use `-no_merged_libraries_hook` to opt out and avoid the deployment target requirement.

---
_Source: WWDC23 Session 10268 page (abstract, chapter summaries, code samples, and resource links)._
