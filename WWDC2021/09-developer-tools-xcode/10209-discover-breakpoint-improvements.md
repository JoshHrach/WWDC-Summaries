# Discover breakpoint improvements
**WWDC21 · Session 10209** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10209/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8 — Xcode 13

## Overview
Xcode 13 brings two significant new breakpoint capabilities: **column breakpoints** for pausing at a specific expression within a line, and **unresolved breakpoint indicators** that visually flag when a breakpoint has not been mapped to any compiled address. Together these address two persistent frustrations: step-in landing on the wrong expression in compound statements, and silent breakpoints that never fire due to typos or unloaded libraries.

The session covers line vs. column breakpoints, symbolic breakpoints with module scoping, the new unresolved breakpoint dashed-icon indicator, and runtime issue breakpoints, with LLDB tips for finding symbol names when a breakpoint fails to resolve.

## Key Topics

### Line Breakpoints — Limitations
- A line breakpoint pauses at the first compiled address on that line.
- When a line contains multiple sub-expressions (chained function calls, closures), the compiler generates multiple line-table entries; stepping in from a line breakpoint may land on the wrong expression due to evaluation order.
- Example: `let vol = fact.convertedToVolume(density: adjustedDensity(fact))` — stepping in lands on `adjustedDensity` before `convertedToVolume`, requiring repeated step-out / step-in cycles.

### Column Breakpoints **[NEW in Xcode 13]**
- Pause at the column of a specific expression, not just the start of the line.
- Set by Command-clicking on any expression in the source editor → "Set Column Breakpoint" in the Actions popover.
- Appear in the source gutter as a small indicator with column info shown in the Breakpoint Navigator subtitle.
- Particularly useful for multi-closure single-line Swift statements (e.g., a `map.filter.forEach` chain) where each closure generates its own line-table entry.
- Disable/enable by clicking the icon; edit by double-clicking; delete by dragging away from the gutter.

### Symbolic Breakpoints — Best Practices
- Set via Breakpoint Navigator → "+" → "Symbolic Breakpoint."
- Match against a function name across all loaded modules; common words (e.g., `toggle`) can match thousands of locations in system libraries.
- **Best practice**: always restrict with a module name (the binary name, e.g., `Fruta`) to limit resolved locations to your own code.
- Use LLDB `image lookup -rn <pattern> <module>` to find the exact spelling of a symbol before creating the breakpoint.

### Unresolved Breakpoint Indicators **[NEW in Xcode 13]**
- When a breakpoint cannot be resolved to any compiled address, Xcode now shows a **dashed/hollow** breakpoint icon instead of the normal solid icon.
- Hovering over the icon shows a tooltip listing common reasons:
  - Symbol name is misspelled or does not exist in the module.
  - The library containing the symbol has not been loaded yet (will auto-resolve when loaded).
  - The source line is inside an `#if` branch that was not compiled (false condition).
  - Debug information was not generated for the module (check Build Settings: `DEBUG_INFORMATION_FORMAT`).
- Replaces the previous silent failure where breakpoints simply never fired.

### Runtime Issue Breakpoints
- Capture runtime issues (e.g., main-thread checker violations, address sanitizer hits) at the moment they occur, instead of viewing them only in the Issue Navigator after the fact.
- Set via Breakpoint Navigator → "+" → "Runtime Issue Breakpoint" → select the type (Main Thread Checker, Address Sanitizer, etc.).
- Some types require enabling the corresponding diagnostic in the scheme editor's Diagnostics tab (accessible via the "Go To" button in the breakpoint editor).
- Allows inspecting live process state at the point of the violation.

### LLDB Tips
- `image lookup -rn <regex> <module>` — search for all symbols matching a regex in a module; useful when a symbolic breakpoint is unresolved due to a spelling error.
- Column PC (introduced in Xcode 11.4): a green underscore under the next expression to execute, distinct from the green line highlight indicating the paused line.

## APIs & Frameworks

**Xcode 13 Debugger**
- Column breakpoints — pause at a specific expression column within a line **[NEW]**
- Unresolved breakpoint dashed icon with tooltip diagnostics **[NEW]**
- Runtime Issue Breakpoints — pause on main-thread checker, address sanitizer, thread sanitizer violations **[existing, improved]**
- Column PC indicator (green underscore) — shows sub-expression paused position **[since Xcode 11.4]**

**LLDB**
- `image lookup -rn <regex> [<module>]` — regex symbol search with optional module scope **[existing]**
- `image lookup` aliased as `module lookup` — binary search within loaded images **[existing]**

**Xcode Scheme Editor**
- Diagnostics tab: Main Thread Checker, Address Sanitizer, Thread Sanitizer toggles — prerequisite for corresponding runtime issue breakpoints **[existing]**

## Code Highlights

Using LLDB to find the correct spelling of a function name when a symbolic breakpoint is unresolved:
```
(lldb) image lookup -rn convert Fruta
```
This lists all symbols in the `Fruta` binary matching "convert" via regex, revealing the correct spelling (e.g., `convertedToMass` instead of `convertToMass`).

Setting a module-scoped symbolic breakpoint to avoid system-library noise:
- Symbol: `toggle`
- Module: `Fruta`
- Result: 3 resolved locations instead of thousands.

## Takeaways
- Column breakpoints eliminate the step-in guessing game for compound expressions and multi-closure lines: Command-click the exact expression you want to pause on.
- The new unresolved breakpoint indicator (dashed icon + tooltip) turns a silent failure into an actionable diagnostic; check spelling, module load order, `#if` conditions, and debug info build settings.
- Always scope symbolic breakpoints to a module name to avoid matching thousands of system library symbols.
- Use `image lookup -rn <pattern> <module>` in LLDB before creating a symbolic breakpoint to confirm the exact symbol name.
- Runtime issue breakpoints let you inspect live state at the moment of a violation rather than working backward from a recorded backtrace.

---
_Source: WWDC21 Session 10209 page (abstract, full transcript, and code samples)._
