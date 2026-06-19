# Link Fast: Improve Build and Launch Times
**WWDC22 · Session 110362** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110362/)

_Platforms:_ iOS 13.4+, iPadOS 13.4+, macOS Ventura 13, tvOS, watchOS

## Overview
This session provides a deep technical dive into both static and dynamic linking — covering what the linker does, recent performance improvements in Apple's static linker (ld64) and dynamic linker (dyld), and practical recommendations for developers to reduce build times and app launch times.

The two headline improvements in Xcode 14 are: ld64 is now up to 2x faster for many projects by exploiting parallelism and algorithmic improvements; and dyld gains a new "page-in linking" feature (iOS 16 / macOS Ventura) that defers fixup application to page fault time, reducing dirty memory and improving launch speed.

## Key Topics
- **Static linking fundamentals** — how the linker evolved from a single-file compiler to multi-file linking; static libraries as ar archives; selective loading (only .o files that resolve undefined symbols are loaded from a static library)
- **ld64 improvements in Xcode 14** — 2x faster for many projects via parallel content copying, parallel LINKEDIT segment construction, parallel UUID and codesigning hash computation, improved algorithms (string_view for exports trie, hardware-accelerated crypto for UUID)
- **Static linker best practices:**
  - Move actively-developed code out of static libraries to avoid full archive rebuilds
  - `-all_load` — forces loading of all .o files from all static libraries, enabling parallel parsing (combine with `-dead_strip` to remove unused code)
  - `-no_exported_symbols` — skips building the exports trie for main app binaries that have no external consumers (saves 2–3 seconds for apps with ~1M exported symbols)
  - `-no_deduplicate` — disables the expensive C++ template deduplication pass; Xcode already enables this for Debug builds by default
- **Dynamic linking fundamentals** — dynamic libraries (dylibs/DSOs/DLLs); how the static linker records symbol promises instead of copying code; ASLR and mmap; fixups (rebases for internal pointers, binds for external symbols); exports trie in LINKEDIT
- **Chained fixups** (new format, requires iOS 13.4+ deployment target) — encodes fixup info directly in DATA pages rather than LINKEDIT, making binaries smaller and enabling page-in linking
- **Page-in linking** (new, iOS 16 / macOS Ventura) — kernel applies DATA page fixups lazily at page-fault time instead of dyld applying all fixups at launch; reduces dirty memory, improves launch time, makes DATA_CONST pages clean/evictable; only works with chained fixups format and only for libraries loaded at launch (not dlopen)
- **dyld shared cache closure caching** (existing since 5 years ago) — caches dylib graph resolution, symbol lookups, and other dyld work across launches when dylibs haven't changed
- **Dynamic linking best practices** — minimize dylib count; avoid I/O and networking in static initializers; find the right balance between static and dynamic libraries; update deployment target to iOS 13.4+ to unlock chained fixups and page-in linking
- **New tooling:**
  - `dyld_usage` (macOS only) — traces dyld activity during app launch including timing per phase
  - `dyld_info` — inspects binaries on disk or in the dyld cache; `-fixup` option shows all fixup locations and targets; `-exports` option shows all exported symbols with offsets

## APIs & Frameworks
These improvements are primarily in the build toolchain and OS runtime, not public APIs. Relevant developer-facing aspects:

**Linker flags (Other Linker Flags in Xcode build settings)**
- `-all_load` — load all .o files from all static libraries (enables parallel parsing)
- `-dead_strip` — remove unreachable code and data; pairs well with `-all_load`
- `-no_exported_symbols` **[NEW consideration]** — skip exports trie generation for main app binaries with no external symbol consumers
- `-no_deduplicate` — disable C++ template dedup pass; already applied automatically by Xcode for Debug builds

**Linker / Build Tools**
- `ld64` (Xcode 14) — up to 2x faster static linker due to parallelism and algorithm improvements **[NEW]**
- Chained fixups format **[NEW]** — smaller LINKEDIT; requires deployment target iOS 13.4+; enabled automatically when targeting iOS 13.4+

**dyld (OS runtime)**
- Page-in linking **[NEW in iOS 16 / macOS Ventura]** — lazy kernel-applied fixups on DATA page fault; reduces dirty memory and launch time; requires chained fixups
- dyld shared cache closure caching — existing; caches dylib graph work across launches

**Command-line tools**
- `dyld_usage` **[NEW]** — traces dyld activity on macOS (also works for Simulator / Mac Catalyst)
- `dyld_info` **[NEW]** — inspects binaries; options: `-fixup`, `-exports`; works for both on-disk files and dyld cache entries

## Code Highlights
No source code changes required. Relevant settings in Xcode "Other Linker Flags":

```
# Load all .o files from static libraries (enables parallel link):
-all_load

# Remove unreachable code (pairs with -all_load):
-dead_strip

# Skip exports trie for main app if nothing resolves symbols from it:
-no_exported_symbols

# Disable C++ template dedup (already default in Debug):
-no_deduplicate
```

Inspect exported symbols count before adding `-no_exported_symbols`:
```bash
dyld_info -exports YourApp.app/YourApp | wc -l
```

Trace dyld activity during launch:
```bash
dyld_usage -f launch YourApp
```

## Takeaways
- ld64 in Xcode 14 is up to 2x faster — simply rebuilding with Xcode 14 improves link time with no project changes.
- Setting deployment target to iOS 13.4+ enables chained fixups (smaller binaries), and iOS 16+ enables page-in linking (faster launch, less dirty memory).
- The three linker flags `-all_load`, `-no_exported_symbols`, and `-no_deduplicate` can meaningfully reduce link time but require evaluating your project's specific characteristics before enabling.
- Too many dylibs slow launch time; too many static libraries slow build time — the Xcode 14 speed improvements may shift your optimal balance toward more static libraries.

---
_Source: WWDC22 Session 110362 page (abstract, chapter summaries, code samples, and resource links)._
