# Profile and optimize your game's memory
**WWDC22 · Session 10106** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10106/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
This session provides a comprehensive guided tour of game memory profiling on Apple platforms, covering how memory is actually measured (virtual vs. physical, dirty vs. clean vs. compressed/swapped), the new Instruments **Game Memory template** for temporal profiling, and a step-by-step memory graph analysis workflow using the `footprint`, `vmmap`, `heap`, `malloc_history`, and `leaks` command-line tools. The session closes with a Metal Debugger Memory Viewer walkthrough for optimizing Metal resource memory.

A key conceptual clarification: "memory footprint" — the primary metric Apple platforms use for memory limit enforcement — equals dirty + compressed/swapped memory and is not the same as virtual address space allocations. On Apple silicon, Metal GPU resources contribute to the same footprint as CPU allocations because CPU and GPU share unified memory.

## Key Topics

### Memory Concepts
- **Virtual memory allocations** — reserved address space; do not immediately consume physical memory
- **Physical memory pages** — 16 KiB each on modern Apple devices; only used pages are counted
- **Dirty pages** — pages written to by the game (heap, modified frameworks, Metal resources on Apple silicon)
- **Compressed/swapped pages** — dirty pages the system has compressed or moved to flash; still charged at uncompressed size
- **Clean pages** — read-only memory-mapped files (textures, audio); can be evicted and reloaded; not counted in footprint
- **Memory footprint** = dirty + compressed/swapped; used for memory limit enforcement
- On Apple silicon: Metal GPU resources (textures, buffers, pipeline state objects) are charged to footprint just like CPU allocations

### Querying Memory Programmatically
- `os_proc_available_memory()` (iOS/iPadOS/tvOS) — available system memory for the process
- `proc_pid_rusage(getpid(), RUSAGE_INFO_CURRENT, ...)` — `ri_phys_footprint` (current), `ri_lifetime_max_phys_footprint` (peak)

### Instruments — Game Memory Template (NEW)
- New **Game Memory** template in Instruments, tailored for Metal games
- **Allocations** instrument — heap allocations, anonymous VM, object reference counts, allocation stack traces; IOAccelerator (Metal resources) and IOSurface (drawables) appear in "All Anonymous VM"
- **Metal Resource Events** instrument — Metal resource allocation/deallocation history with labels; density tracks per device
- **VM Tracker** instrument — dirty vs. compressed/swapped memory footprint breakdown; maps memory-mapped resource files
- `xctrace record --template "Game Memory" --attach <process> --output <file> --time-limit <time>` — command-line recording; supports `--device-name` for remote devices

### Memory Graph Analysis Workflow
1. Enable **Malloc Stack Logging** in Xcode Scheme → Run → Diagnostics (Live Allocation Only recommended)
2. Capture memory graph: click debug memory graph button in Xcode, or `leaks <PID> --outputGraph foo.memgraph` (Mac only, supports remote SSH)
3. Analyze with CLI tools:
   - `footprint foo.memgraph` — high-level category breakdown; IOAccelerator, heap MALLOC_ pools, untagged VM_ALLOCATE
   - `vmmap --summary foo.memgraph` — dirty vs. swapped sizes per region; heap zones; virtual sizes
   - `vmmap foo.memgraph` (standard mode) — line-by-line per VM region
   - `heap --quiet --sortBySize --showSizes foo.memgraph` — malloc'd objects by class, sorted by total size; improved type identification (uses MallocStackLogging caller info)
   - `heap --address "ClassName.*" -size "[10000000-]" foo.memgraph` — find object addresses by class + size filter
   - `malloc_history --callTree --invert foo.memgraph <address>` — allocation call stack for a specific object
   - `leaks --traceTree <address> foo.memgraph` — reference tree for an object
4. **Xcode Memory Debugger** (Xcode 14 redesigned) — bidirectional graph view showing ingoing and outgoing references, neighbor selection popover for large graphs

### Custom Memory Tags for VM_ALLOCATE
- Up to 16 app-specific tags via `VM_MAKE_TAG(VM_MEMORY_APPLICATION_SPECIFIC_1..16)`
- Apply in `mmap()` as the `fd` argument (instead of -1) or as flags in `mach_vm_allocate()`
- Tags appear as named categories in `footprint` and `vmmap` output instead of "untagged VM_ALLOCATE"

### Metal Debugger — Memory Viewer
- Access via GPU Frame Capture → Summary → Show Memory button
- **Insights column** — per-resource memory saving suggestions
- **Allocated Size** — sort to find largest resources; audit for oversized assets
- **Time Since Last Bound** — identify unused/infrequently used resources; candidates for release or purgeable-volatile marking
- **Purgeable state** — `MTLResource.setPurgeableState(.volatile)` lets Metal evict resource under memory pressure; check state before use and reload if `.empty`
- **Pixel Format column** (right-click to show) — identify candidates for 16-bit half-precision formats, single-channel alpha, ASTC/BC block compression
- **Lossy compression** (A15 Bionic+) — reduce texture and render target memory while preserving quality
- **Storage mode optimization**:
  - `.memoryless` — single-pass temporary render targets (depth, stencil, MSAA); no memory allocated, no bandwidth used
  - `.private` — GPU-only resources
  - `.managed` — not needed on Apple silicon (unified memory)
- **Aliased resources from MTLHeap** — share memory backing for resources not used simultaneously; requires careful synchronization

## APIs & Frameworks

**System / C APIs**
- `os_proc_available_memory()` — available system memory (iOS/tvOS/watchOS)
- `proc_pid_rusage(pid, RUSAGE_INFO_CURRENT, buffer)` — `rusage_info_v6`
  - `ri_phys_footprint: uint64_t` — current memory footprint
  - `ri_lifetime_max_phys_footprint: uint64_t` — peak footprint
- `VM_MAKE_TAG(VM_MEMORY_APPLICATION_SPECIFIC_1)` — custom memory region tag
- `mmap(..., tag, ...)` — `mmap` with custom tag as file descriptor
- `mach_vm_allocate(..., VM_FLAGS_ANYWHERE | VM_MAKE_TAG(...))` — allocate with custom tag

**Metal**
- `MTLResource.setPurgeableState(_:)` — `.nonVolatile`, `.volatile`, `.empty`
- `MTLTextureDescriptor.storageMode` — `.memoryless`, `.private`, `.shared`, `.managed`
- `MTLHeap` — allocate aliased resources from a shared memory pool

**Instruments (Xcode 14)**
- Game Memory template — **[NEW]**
- Allocations instrument
- Metal Resource Events instrument — **[NEW]**
- VM Tracker instrument
- Virtual Memory Trace instrument
- Metal Application / GPU instruments

**Command-line tools**
- `xctrace` — `--template`, `--attach`, `--device-name`, `--output`, `--time-limit`
- `footprint <memgraph>` — memory category breakdown
- `vmmap [--summary] <memgraph>` — virtual memory region analysis
- `heap [--quiet] [--sortBySize] [--showSizes] [--address <pattern> -size <range>] <memgraph>` — heap object analysis; improved type ID **[NEW in Xcode 14]**
- `malloc_history [--callTree] [--invert] <memgraph> <address>` — allocation backtraces
- `leaks [--traceTree <address>] [--outputGraph <file>] <PID>` — leak detection and reference trees

## Code Highlights

Query current and peak memory footprint:
```c
rusage_info_current rusage_payload;
int ret = proc_pid_rusage(getpid(), RUSAGE_INFO_CURRENT, (rusage_info_t *)&rusage_payload);
uint64_t footprint = rusage_payload.ri_phys_footprint;
uint64_t peak      = rusage_payload.ri_lifetime_max_phys_footprint;
```

Tag custom VM_ALLOCATE region with app-specific tag:
```c
int tag = VM_MAKE_TAG(VM_MEMORY_APPLICATION_SPECIFIC_1);
void *reservation = mmap(NULL, length, PROT_READ | PROT_WRITE,
                          MAP_ANONYMOUS | MAP_PRIVATE, tag, 0);
```

Record Instruments trace from the command line:
```sh
xctrace record --template "Game Memory" \
               --attach ModernRenderer \
               --output ModernRenderer.trace \
               --time-limit 30s
```

## Takeaways
- Memory footprint (dirty + compressed/swapped pages) — not virtual address space — is what Apple platforms measure and enforce; on Apple silicon, Metal GPU resources count toward the same footprint.
- The new Instruments Game Memory template (Xcode 14) combines Allocations, Metal Resource Events, and VM Tracker for a one-stop temporal memory profiling workflow; use `xctrace` to automate trace capture.
- The memory graph + CLI tool workflow (`footprint` → `vmmap` → `heap` → `malloc_history` → `leaks`) provides layered analysis from category breakdown to individual object allocation history and reference trees; always enable Malloc Stack Logging before capturing a memory graph.
- Metal Debugger Memory Viewer identifies resource optimization opportunities (purgeable state, pixel format, storage mode, aliased heaps) that can significantly reduce game memory footprint.

---
_Source: WWDC22 Session 10106 page (abstract, chapter summaries, code samples, and resource links)._
