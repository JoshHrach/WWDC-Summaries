# What's New in Clang and LLVM
**WWDC19 · Session 409** · [Watch](https://developer.apple.com/videos/play/wwdc2019/409/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6

## Overview
Clang and LLVM in Xcode 11 ship with improvements across five areas: Bitcode-powered 64-bit watchOS support enabling seamless Series 4 app compatibility, a new `-Oz` optimization level for aggressive code size reduction using machine-level function outlining, four language-level code size improvements (block metadata merging, NSObject direct IVAR access, C++ STL de-inlining, and static destructor suppression), five new default compiler warnings for C, Objective-C, and C++, and three new static analyzer checks covering use-after-move, dangling C string pointers, and reference counting bugs in DriverKit and IOKit.

## Key Topics

**Bitcode and watchOS 64-bit Transition**
Apple Watch Series 4 uses a fully 64-bit chip, but App Store apps were 32-bit. Bitcode — an intermediate LLVM IR preserved in the binary — allowed the App Store to recompile existing 32-bit apps for the 64-bit architecture on day one, without developer resubmission. For watchOS 6, the compiler also collects 64-bit Bitcode to maximize optimization opportunities specific to the new architecture.

**-Oz: New Code Size Optimization Level**
`-Oz` (new in Xcode 11) prioritizes code size above all other metrics, including execution speed. Its primary mechanism is Machine IR function outlining: sequences of identical instructions across multiple functions are extracted into a new shared function and replaced with calls/branches. Demonstrated up to 25% binary size reduction on test programs. Trade-off: execution time may increase (added call overhead); LLDB stack traces may show synthesized `outlined_function_*` frames. Not recommended for performance-critical code; profile with Instruments first to identify hot paths. Use `-Os` (default) for balanced size/speed; `-O3` for maximum speed.

**Language-Level Code Size Improvements**
Four optimizations enabled by default in Xcode 11:
1. *Objective-C block metadata merging*: blocks with identical capture layouts share synthesized copy/destroy helper functions; 2–7% size reduction in ObjC apps.
2. *NSObject direct IVAR access*: classes deriving directly from NSObject have known-stable ABI, allowing the compiler to hardcode IVAR offsets instead of dynamic table lookups; ~2% size reduction.
3. *C++ STL de-forced-inlining*: libc++ methods like `std::vector::push_back` are no longer force-inlined, allowing the optimizer to decide; up to 7% size reduction in release builds with heavy STL use, plus improved debuggability (breakpoints on STL calls now land correctly).
4. *C++ static destructor suppression*: new `[[clang::no_destroy]]` attribute prevents global objects from registering destructors that are meaningless on iOS/iPadOS (where the app lifecycle has no clean shutdown); ~1% size reduction. Can also be applied project-wide via a build setting.

**New Compiler Diagnostics (all on by default in Xcode 11)**
- *Call to pure virtual from constructor/destructor*: calling a pure virtual method in a base class ctor/dtor is undefined behavior — the vtable slot is empty.
- *Memset transposed arguments*: detects when the size and value arguments to `memset` are reversed.
- *Return of `std::move` pessimization*: warns when `return std::move(x)` prevents mandatory copy elision (NRVO) and returns a slice of a derived type.
- *sizeof-pointer-div*: detects the classic `sizeof(ptr) / sizeof(ptr[0])` pattern that returns pointer size (not array size) after an array parameter decays to a pointer.
- *Defaulted function deleted*: explains exactly why `= default` cannot be synthesized (e.g., a member has a reference type).

**New Static Analyzer Checks**
- *Use-after-move*: reports code that reads from a moved-from variable, which is in an unspecified state.
- *Dangling C string from `std::string`*: detects when `c_str()` is called on a temporary or local `std::string` that goes out of scope before the returned pointer is used; fix by extending the `std::string` lifetime.
- *DriverKit / IOKit reference counting*: enforces the naming-convention-based retain/release rules (methods return +1 by default; getters return +0); detects leaks and over-releases; supports `DRIVERKIT_RETURNS_NOT_RETAINED` annotation to document intentional convention deviations.

## APIs & Frameworks

**Clang Compiler Flags**
- `-Oz` **[NEW]** — optimize for code size aggressively (outlining, above `-Os`)
- `-Os` — default Xcode optimization level; balanced size/speed
- `-O3` — optimize for maximum execution speed
- PGO (Profile-Guided Optimization) — collect runtime profiles to guide re-compilation
- LTO (Link-Time Optimization) — cross-file inlining and outlining; improves with `-Oz`

**Clang Language Extensions**
- `[[clang::no_destroy]]` **[NEW]** — suppress static/global destructor registration for a variable
- `DRIVERKIT_RETURNS_NOT_RETAINED` **[NEW]** — annotation for IOKit/DriverKit methods returning +0 that would otherwise violate the default +1 convention

**Static Analyzer**
- Use-after-move check **[NEW]** — catches reads from moved-from C++ objects
- Dangling-`c_str` check **[NEW]** — catches `std::string::c_str()` lifetime mismatches
- IOKit/DriverKit retain/release check **[NEW]** — enforces +1/+0 naming conventions; detects leaks and over-releases

**Xcode 11 Build Settings**
- Optimization Level: `-Oz` option added
- Per-file compiler flags in Build Phases → Compile Sources
- Analyze During Build — run static analyzer on every build

**Command-line Tools**
- `size -l -m <binary>` — per-section binary size breakdown; use to measure `__TEXT,__text` (executable instructions)

## Code Highlights

Enabling `-Oz` and measuring size impact:

```bash
# In Build Settings: Optimization Level → Optimize for Size [-Oz]
# Or per-file in Compile Sources → Compiler Flags: -Oz

# Measure __text section size before/after:
size -l -m MyApp.app/MyApp
```

Using `[[clang::no_destroy]]` to suppress a global destructor:

```cpp
// Before: destructor runs at (undefined) app termination time
static Logger globalLogger;

// After: no destructor registered; flush manually in app lifecycle callbacks
[[clang::no_destroy]] static Logger globalLogger;
```

Fixing a dangling `c_str()` pointer:

```cpp
// BAD: std::string destroyed before pointer is used
const char* generateGreeting(const char* name) {
    std::string greeting = std::string("Hello, ") + name;
    return greeting.c_str();  // UB: greeting destroyed here
}

// GOOD: extend lifetime by returning std::string; call c_str() at point of use
std::string generateGreeting(const char* name) {
    return std::string("Hello, ") + name;
}
// Caller: auto s = generateGreeting(name); use(s.c_str());
```

IOKit retain/release annotation:

```cpp
// Convention: method returning IOService* defaults to +1 (retained)
// Rename to getFirstDevice (getter → +0) OR annotate:
IOService* findFirstDevice() DRIVERKIT_RETURNS_NOT_RETAINED {
    return array->getObject(0);  // getObject is +0 (getter)
}
```

## Takeaways
- `-Oz` delivers meaningful binary size reductions (up to 25% in outlined code) at a potential execution-time cost; use Instruments to identify hot paths before applying it project-wide.
- The four language-level size optimizations are automatic in Xcode 11 — no code changes required; expect 2–7% smaller binaries for Objective-C apps.
- The five new default warnings catch real bugs (transposed `memset` args, UB pure-virtual calls, `sizeof`-pointer mistakes) — fix them before they reach production.
- Run the static analyzer regularly (`Product → Analyze` or enable Analyze During Build); the new use-after-move and dangling-pointer checks catch correctness issues that unit tests rarely exercise.
- If you write IOKit or DriverKit code, immediately enable the static analyzer's reference counting checks; manual retain/release bugs are among the hardest crashes to reproduce.

---
_Source: WWDC19 Session 409 page (abstract, transcript, and resource links)._
