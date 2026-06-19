# Advanced Debugging and the Address Sanitizer
**WWDC15 · Session 413** · [Watch](https://developer.apple.com/videos/play/wwdc2015/413/)

_Platforms:_ iOS 9, OS X El Capitan 10.11

## Overview
This session covers advanced debugging techniques in Xcode 7, including the enhanced View Debugger, sophisticated breakpoint actions, and the brand-new Address Sanitizer (ASan). The first half demonstrates how to diagnose AutoLayout constraint conflicts and runtime exceptions using Xcode's debugging UI, while the second half introduces Address Sanitizer as a powerful memory-corruption detection tool.

Address Sanitizer is an LLVM-based tool for C-family languages that detects buffer overflows, use-after-free errors, heap/stack/global variable misuse, and more — all at runtime with much lower overhead than Guard Malloc. Unlike Guard Malloc, it also works on iOS devices and integrates directly into the Xcode debugger UI with rich diagnostics.

The session uses a demo fitness app called "Jogr" to walk through real-world bugs: a clipped UI element due to a spurious AutoLayout constraint, an out-of-bounds string range exception caught via exception breakpoints, and a heap buffer overflow in MapKit route rendering caught by Address Sanitizer.

## Key Topics

### View Debugger
- Double-click a view in the 3D canvas to **focus** on a subtree, hiding unrelated hierarchy.
- Size Inspector shows active vs. inactive constraints at runtime, making it easy to spot conflicting or redundant constraints.
- Debug Navigator mirrors the constraint hierarchy as a tree for quick navigation.

### Advanced Breakpoints
- **Exception Breakpoints**: Configured to stop on Objective-C exceptions only; used to pause at the exact throw site rather than at `main`.
- **LLDB expression in breakpoint actions**: `po $arg1` (print the first argument of `objc_exception_throw`) reveals the exception message without any source changes.
- **Print-and-continue breakpoints**: Logging breakpoints with "Automatically continue after evaluating actions" checked replace ad-hoc `NSLog`/`print` calls.

### Address Sanitizer (ASan)
- Enabled per-scheme in **Edit Scheme → Diagnostics → Enable Address Sanitizer**; requires recompilation.
- Detects: heap buffer overflows/underflows, stack buffer overflows, global variable overflows, use-after-free, use-after-return, use-after-scope, double-free, invalid free, C++ container overflow (`std::vector` out-of-bounds).
- Integrates diagnostics (stack trace + heap object history) directly into the Xcode debugger UI.
- Works on iOS devices — Guard Malloc does not.
- Typical runtime overhead: ~2× CPU, 2–3× memory.
- Recommended for continuous integration: pass `-enableAddressSanitizer YES` to `xcodebuild`.

### How ASan Works
- Compiler (`clang -fsanitize=address`) instruments every memory access with a shadow-memory check.
- Shadow memory: 1 byte tracks 8 bytes of real memory; reserved (not fully allocated) at process launch.
- Custom `malloc` replaces the system allocator: inserts poisoned **red zones** around allocations, delays free'd memory reuse, records alloc/dealloc stack traces.
- Stack variables: red zones inserted between locals at compile time, poisoned on function entry, un-poisoned on exit.
- Global variables: red zones added around them at compile time.
- Standard library functions (e.g., `memcpy`) are intercepted via function interposition even without recompilation.

### Complementary Tools Comparison
- **Guard Malloc**: no recompilation needed, but no iOS device support; misses some off-by-one overflows.
- **NSZombie**: catches Objective-C over-releases; use Zombies Instrument for full power.
- **Malloc Scribble**: fills allocated/freed memory with preset constants to expose uninitialized-memory bugs.
- **Leaks Instrument**: finds retain cycles and abandoned memory.

## APIs & Frameworks

- `Xcode View Debugger` — **[NEW]** focus mode via double-click; runtime constraint inspection **[NEW]**
- `Debug View Hierarchy` button in the Debug bar
- `Size Inspector` — runtime constraint display with active/inactive state **[NEW]**
- `Breakpoint Navigator` — exception breakpoints, action breakpoints
- `LLDB` — `po $arg1` expression to print `objc_exception_throw` argument
- `Address Sanitizer` (`-fsanitize=address` / `clang` flag) **[NEW]**
- `asan` runtime dylib **[NEW]**
- Edit Scheme → Diagnostics tab → **Enable Address Sanitizer** checkbox **[NEW]**
- `xcodebuild -enableAddressSanitizer YES` **[NEW]**
- `MKPolyline` / `+polylineWithPoints:count:` (MapKit) — demonstrated bug site
- `MKMapPoint` struct (MapKit)
- `NSRange` / `NSMakeRange` — demonstrated out-of-bounds range bug
- `Guard Malloc` (libgmalloc)
- `NSZombie`
- Malloc Scribble
- Leaks Instrument

## Code Highlights

Breakpoint LLDB action to print the in-flight Objective-C exception:
```
po $arg1
```

Incorrect buffer size calculation (the bug ASan caught):
```objc
// BUG: sizeof(MKMapPoint *) — pointer size, not struct size
NSUInteger count = bufferSize / sizeof(MKMapPoint *);

// FIX: sizeof(MKMapPoint) — full struct with two doubles
NSUInteger count = bufferSize / sizeof(MKMapPoint);
```

## Takeaways
- The View Debugger's focus mode and runtime constraint inspector dramatically speed up AutoLayout debugging.
- Exception breakpoints with `po $arg1` give you the exception message at the throw site — no source changes needed.
- Address Sanitizer is the go-to tool for memory-corruption bugs on both simulator and iOS device; enable it in CI for maximum coverage.
- Off-by-one pointer/struct size errors (`sizeof(T*)` vs. `sizeof(T)`) are exactly the kind of subtle bug ASan catches where code inspection alone fails.

---
_Source: WWDC15 Session 413 page (abstract, chapter summaries, code samples, and resource links)._
