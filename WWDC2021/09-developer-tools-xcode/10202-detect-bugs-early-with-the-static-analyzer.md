# Detect bugs early with the static analyzer
**WWDC21 · Session 10202** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10202/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
The Xcode static analyzer performs compile-time analysis of C, C++, and Objective-C code—including mixed Swift/Objective-C projects—to surface bugs before the app is ever run. Unlike compiler warnings, the analyzer traces complete code paths, catching issues in rare branches that tests might never exercise. Running it requires a single menu action: Product → Analyze (Command-Shift-B).

Xcode 13 adds several new static analysis checks, notably for infinite loops, redundant code, side effects inside assertions, and C++ move/forward operator misuse. These checks include open-source contributions to Apple's Clang compiler. The session also covers customization options: enabling analysis during every build, choosing shallow vs. deep mode, enabling or disabling individual check categories, and analyzing a single file in isolation.

## Key Topics

### What the Static Analyzer Does
- Analyzes source code statically (no execution required) to find bugs that runtime testing might miss.
- Works on C, C++, and Objective-C code in pure or mixed Swift/Objective-C projects.
- Categories of bugs found: security issues, logical bugs, API misuse, null-value returns from non-null–annotated methods, and more.
- Produces issue reports with annotated arrows in the editor showing the sequence of events leading to a bug—readable from bottom to top.

### Running the Analyzer
- **One-shot**: Product → Analyze (Cmd-Shift-B) — analyzes all files in the active scheme's targets.
- **Per-build**: Enable "Analyze During 'Build'" in Build Settings; only modified files are re-analyzed (like incremental builds).
- **Single file**: Product → Perform Action → Analyze file — fast check of a specific file without a full build, useful for header changes.

### New Checks in Xcode 13

#### Side Effects in Assertions **[NEW]**
- Detects when `NSAssert`, C assert, or C++ assert contains expressions with side effects (e.g., incrementing a counter inside the assert condition).
- Asserts may be disabled in Release builds, so such side effects would be silently dropped in production.
- Fix: move the side-effecting expression outside the assert.

#### Infinite Loop Detection **[NEW]**
- Detects loops where the loop counter is never modified—e.g., incrementing the wrong variable inside a nested loop.
- Reports with an explanation of why the loop never terminates.

#### Redundant Code / Dead Code **[NEW]**
- Detects unnecessary branch conditions (conditions that are always true or always false) and dead code paths.

#### C++ Move and Forward Operator Errors **[NEW]**
- Detects incorrect use of `std::move` and `std::forward` in C++ code.

### Analysis Modes
- **Deep mode** (default): traces bugs across multiple function calls for maximum coverage; slower.
- **Shallow mode**: analyzes within single functions only; faster, suitable for projects sensitive to build time.

### Customizing Checks
- In Build Settings, search "analysis" to filter relevant options.
- Enable or disable individual check categories (e.g., enable security checks for security-critical code; disable irrelevant checks to reduce noise).
- Available categories include: security, logic errors (nullability, dead code, infinite loops), API misuse, and more.

### Practical Workflow
1. Run analyzer regularly during development to catch issues early.
2. Enable "Analyze During Build" for continuous analysis on modified files.
3. Use the visual bug report (arrows in editor) to trace the event sequence leading to each issue.
4. Fix the issue, re-run analyzer to confirm resolution.

## APIs & Frameworks

**Xcode Static Analyzer (Clang)**
- Nullability violation detection: catches Objective-C methods returning `nil` despite being annotated `nonnull` **[existing]**
- API misuse checks **[existing]**
- Security issue checks (memory safety, format strings, etc.) **[existing]**
- Side-effect-in-assert check (`NSAssert`, C assert, C++ assert) **[NEW in Xcode 13]**
- Infinite loop detection **[NEW in Xcode 13]**
- Redundant/dead-code detection **[NEW in Xcode 13]**
- C++ `std::move` / `std::forward` misuse check **[NEW in Xcode 13]**

**Build Settings (Xcode)**
- `CLANG_STATIC_ANALYZER_MODE` — "shallow" or "deep" **[existing]**
- "Analyze During 'Build'" (`RUN_CLANG_STATIC_ANALYZER`) — runs analyzer on every build, incremental **[existing]**
- Per-category check enable/disable settings (e.g., `CLANG_ANALYZER_SECURITY_INSECUREAPI_RAND`) **[existing]**

**Objective-C Nullability Annotations**
- `NS_RETURNS_NOT_RETAINED`, `nonnull`, `NS_ASSUME_NONNULL_BEGIN/END` — annotate return value expectations enforced by the analyzer **[existing]**

## Code Highlights

Example of a side-effect bug inside `NSAssert` (incorrect):
```objc
// BUG: counter++ may be stripped in Release builds
NSAssert(counter++ <= totalPlanets, @"Too many moons");
```

Fixed version — side effect moved outside the assert:
```objc
counter++;
NSAssert(counter <= totalPlanets, @"Too many moons");
```

Example of an infinite loop bug (incorrect):
```objc
// BUG: value incremented instead of column — loop never advances
for (int column = 0; column < width; value++) {
    grid[row][column] = value;
}
```

Fixed:
```objc
for (int column = 0; column < width; column++) {
    grid[row][column] = value;
}
```

## Takeaways
- The Clang static analyzer is a zero-run-time-cost bug finder that catches issues—especially in rare code paths—before testing or shipping.
- Xcode 13 adds four new check categories: side effects in asserts, infinite loops, redundant code, and C++ move/forward errors.
- Enable "Analyze During Build" for continuous analysis on modified files at near-zero overhead compared to a full analyze pass.
- Use shallow mode in build-time-sensitive projects; enable individual check groups (e.g., security) strategically rather than accepting all or none.

---
_Source: WWDC21 Session 10202 page (abstract, full transcript, and resource links)._
