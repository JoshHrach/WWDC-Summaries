# Port your Mac app to Apple silicon
**WWDC20 · Session 10214** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10214/)

_Platforms:_ macOS Big Sur 11 (Apple silicon / arm64)

## Overview
This session guides experienced macOS developers through the process of recompiling their existing Intel-only Mac apps as universal binaries that run natively on Apple silicon (arm64). It covers the full journey: building universal apps in Xcode 12, fixing compile-time and link-time portability issues, runtime testing and debugging on Apple silicon, handling in-process and out-of-process plug-ins, and practical tips for distributing universal apps.

The session emphasizes that building natively is often as simple as clicking Run in Xcode 12, with app-specific portability issues typically falling into a small number of well-defined categories: incorrect target conditionals, CPU-specific code, non-universal binary dependencies, and low-level API behavior differences between arm64 and x86_64.

## Key Topics

### Building Universal Apps
- Universal (fat) binaries contain code for multiple CPU architectures; from 2020, Mac apps should target both `arm64` (Apple silicon) and `x86_64` (Intel)
- Xcode 12: run destination menu offers "My Mac" (native arm64), "My Mac (Rosetta)" (Intel under Rosetta), and "My Mac (Designed for iPad)" for iOS apps
- Cross-compilation is fully supported — any Intel Mac running Xcode 12 can build arm64 code
- Rosetta runs entire processes translated (cannot mix native and translated code in one process); does not support kernel extensions, AVX instructions, or virtualization

### Build-Time Issues
- **`PAGE_SIZE` macro**: no longer a compile-time constant on Apple silicon; replace with `PAGE_MAX_SIZE` (compile-time upper bound) or `vm_page_size` (runtime value)
- **Target conditionals**: do not equate `__x86_64__` with macOS; use `TARGET_OS_OSX` / `#if os(macOS)` for platform checks; use `TARGET_CPU_X86_64` / `#if arch(x86_64)` for architecture checks
- **CPU-specific assembly/intrinsics**: guard all x86 inline assembly and SSE/AVX code with architecture conditionals; provide arm64 equivalents or replace with Accelerate/Compression framework calls
- **Non-universal binary dependencies**: use `lipo -info <file>` to check architectures; contact vendors for universal versions; scan for `.a`, `.dylib`, `.framework`, `.xcframework` files

### Runtime Portability Issues
- **`mach_absolute_time`**: returns ticks (not nanoseconds); the tick-to-nanosecond ratio differs by architecture (arm64 tick ≈ 1ns but do not assume); use `clock_gettime_nsec_np(CLOCK_UPTIME_RAW)` for guaranteed nanosecond timestamps
- **Memory ordering**: arm64 has a weaker memory ordering model than x86_64; data races that appear benign on Intel may crash on Apple silicon; use Thread Sanitizer to detect race conditions
- **Asymmetric CPU cores**: Apple silicon has P-cores (high-performance) and E-cores (energy-efficient); both can be active simultaneously; avoid spinlocks and busy-waiting which waste P-core time
- Replace spinlocks with `os_unfair_lock`; replace busy-wait loops with `NSCondition` or `pthread_cond_wait`; prefer GCD; split work into smaller units rather than splitting by CPU count

### Testing and Debugging
- Native arm64 testing requires an Apple silicon Mac; all other development (building, cross-compilation) works on any Mac
- Run test suites in both native (`arch=arm64`) and Rosetta (`arch=x86_64`) modes with `xcodebuild -destination`
- Archive builds in Xcode 12 produce universal apps by default
- Xcode Organizer shows CPU architectures in archives; crash logs now indicate CPU architecture and whether the process ran translated

### Plug-ins
- **Standard NSExtension plug-ins**: both native and translated plug-ins are supported automatically
- **In-process plug-ins** (loaded via `dlopen` / `Bundle.load`): all code in one process must share the same CPU architecture; native apps can only load native plug-ins; Rosetta apps can only load Intel plug-ins
- **Out-of-process plug-ins** (XPC): no CPU architecture restriction; preferred for security, stability, and flexibility
- Build first-party plug-ins as universal; for third-party Intel-only in-process plug-ins, users can force Rosetta via "Open using Rosetta" in Finder (can be disabled via Info.plist key)
- Always check `dlopen` return value and call `dlerror()` on failure to diagnose architecture mismatch errors

## APIs & Frameworks

- **Xcode 12** — run destination selector: "My Mac", "My Mac (Rosetta)"; universal binary support **[NEW]**; Organizer architecture info **[NEW]**
- `lipo -info <file>` — inspect binary CPU architectures (command-line tool)
- `xcodebuild -destination arch=arm64` / `arch=x86_64` — CI/command-line build targeting
- `PAGE_MAX_SIZE` — compile-time upper bound for page size (replaces `PAGE_SIZE` constant)
- `vm_page_size` — runtime page size query
- Target conditionals: `TARGET_OS_OSX`, `TARGET_OS_SIMULATOR`, `TARGET_CPU_X86_64`, `TARGET_CPU_ARM64`
- Swift equivalents: `#if os(macOS)`, `#if arch(x86_64)`, `#if arch(arm64)`
- `mach_absolute_time()` — tick-based monotonic time (not nanoseconds; do not assume 1ns per tick)
- `clock_gettime_nsec_np(CLOCK_UPTIME_RAW)` **[KEY replacement]** — guaranteed nanosecond monotonic time
- **Accelerate** framework — cross-platform optimized math (replaces SSE/AVX intrinsics)
- **Compression** framework — cross-platform compression (replaces platform-specific algorithms)
- `os_unfair_lock` / `os_unfair_lock_lock` / `os_unfair_lock_unlock` — blocking lock (replaces spinlocks)
- `NSLock`, `NSCondition` — blocking synchronization primitives
- `pthread_mutex_t`, `pthread_cond_t` — POSIX blocking mutex and condition variables
- Grand Central Dispatch (`DispatchQueue`) — preferred concurrency primitive
- `dlopen(_:_:)` / `dlerror()` — dynamic plug-in loading; always check return value
- `Bundle.load()` — Swift plug-in loading
- XPC (`NSXPCConnection`, `xpc_connection_create`) — out-of-process plug-in model
- **Thread Sanitizer** (Xcode scheme Diagnostics) — detect data races across architectures
- `Info.plist` key to disallow "Open using Rosetta" checkbox

## Code Highlights

Wrong (assumes ticks = nanoseconds):
```swift
let ticks = mach_absolute_time()
let seconds = Double(ticks) / 1_000_000_000
```

Correct (nanoseconds guaranteed):
```swift
let nanoseconds = clock_gettime_nsec_np(CLOCK_UPTIME_RAW)
let seconds = Double(nanoseconds) / 1_000_000_000
```

Prefer blocking over spinlock/busy-wait:
```swift
// Instead of spinlock:
os_unfair_lock_lock(&lock)
performWork()
os_unfair_lock_unlock(&lock)

// Instead of busy-wait:
condition.lock()
while !taskQueue.hasAnyWork { condition.wait() }
let task = taskQueue.pop()
condition.unlock()
```

Check dlopen result for architecture mismatch diagnosis:
```c
void *plugin = dlopen("./path/to/plugin.dylib", RTLD_NOW);
if (plugin == NULL) {
    fprintf(stderr, "loading failed: %s\n", dlerror());
}
```

## Takeaways

- Building universal for Apple silicon is often a single click in Xcode 12; use `lipo -info` to find non-universal binary dependencies (the most common blocker), then request universal builds from vendors.
- Replace architecture-based `#if __x86_64__` guards with semantic target conditionals (`TARGET_OS_OSX`, `TARGET_CPU_X86_64`) and replace `mach_absolute_time()` with `clock_gettime_nsec_np(CLOCK_UPTIME_RAW)`.
- Eliminate spinlocks and busy-waiting — on Apple silicon's asymmetric cores they waste P-core time and degrade performance; prefer `os_unfair_lock`, `NSCondition`, or GCD.
- Run test suites under both native arm64 and Rosetta destinations; use Thread Sanitizer to catch data races that may be latent on x86 but crash on arm64.

---
_Source: WWDC20 Session 10214 page (abstract, chapter summaries, code samples, and resource links)._
