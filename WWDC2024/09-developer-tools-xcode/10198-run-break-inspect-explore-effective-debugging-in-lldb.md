# Run, Break, Inspect: Explore Effective Debugging in LLDB
**WWDC24 · Session 10198** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10198/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, visionOS 2, watchOS 11

## Overview
This session teaches a structured approach to debugging using LLDB as the primary tool. The core model — Run, Break, Inspect — frames debugging as a search problem: the bug exists somewhere between program start and the observed failure, and each inspection narrows the search space. The session covers crashlog analysis, backtrace navigation, breakpoint types, breakpoint actions, high-firing breakpoints, the `p` command, and the new `@DebugDescription` macro from Swift 6.

The session is practical and demonstration-driven, using the "Destination Video" multiplatform app as a running example. It emphasizes that LLDB lets developers inspect and evaluate expressions without recompiling — a significant productivity advantage over print-debug cycles.

## Key Topics

**Debugging as a Search Problem**
The bug lies between program start and the incorrect behavior. Each variable inspection or breakpoint stop brings developers closer to the root cause. LLDB's backtrace, variable viewer, breakpoints, and expression evaluator are the four core search tools.

**Crashlogs & Backtraces**
Crashlogs can be opened directly in Xcode (secondary-click → Open With → Xcode, then choose a project for symbolication context). LLDB reconstructs the debugging session at crash time. The Debug Navigator shows the full backtrace. Correct symbolication requires a matching source checkout and the dSYM bundle.

**Breakpoints**
A single line breakpoint in SwiftUI declarative code can resolve to multiple locations (e.g., constructor call site, trailing closure, action closure). `breakpoint list` shows all resolved locations with IDs (e.g., 1.1, 1.2, 1.3). Individual locations can be disabled. Command shortcuts: `b <file:line>` sets a breakpoint; `f <N>` selects a backtrace frame.

**Breakpoint Actions**
Breakpoints can run LLDB commands automatically on each hit — including `p` expressions, `tbreak` to plant one-shot breakpoints, or `continue` to resume automatically. Configured via Xcode's Edit Breakpoint UI or `break command add` on the CLI. This enables log-like output without recompiling.

**High-Firing Breakpoints**
Three strategies for breakpoints that fire too often:
1. **Condition** — `break modify --condition "video.duration > 60"` stops only when the expression is true.
2. **Temporary breakpoint via action** — auto-continue breakpoint at A plants a `tbreak` at B; B only fires once, after A was reached.
3. **Ignore count** — `break modify --ignore-count 10` skips the first N hits.
For extremely high-frequency sites, use `raise(SIGSTOP)` inside an `if` statement in source code — the debugger takes over as if a breakpoint was hit, with no per-iteration overhead.

**The `p` Command**
Since Xcode 15, `p` is the "do what I mean" print command — a unified alias that replaces `po`, `p`, `v`, and `e` for most use cases. It can inspect variables, evaluate complex Swift expressions, and operates in any backtrace frame. Results can be built incrementally without recompiling. Also available: the Xcode variable viewer, hover-over variable inspection, and QuickLook button for rich types.

**Swift Error Breakpoint**
A Swift Error breakpoint pauses execution the moment any Swift error is thrown — useful for catching which `try` statement in a complex initializer is failing, without setting a breakpoint on every possible line.

**@DebugDescription Macro (Swift 6)**
New Swift 6 macro that lets types define a concise debugger summary directly in source. Annotate the type with `@DebugDescription` and provide a `var debugDescription: String` computed property using string interpolation and stored properties. The summary then appears in both the `p` command output and the Xcode variable viewer. Supersedes `CustomDebugStringConvertible` for most cases (those using only interpolation + stored properties).

## APIs & Frameworks

**LLDB**
- `p` — "do what I mean" print / expression evaluator (reworked in Xcode 15)
- `po` — legacy print-object (still valid; use `p` instead for most cases)
- `b <file:line>` — set breakpoint (alias for `breakpoint set`)
- `breakpoint list` — show all breakpoints and their resolved locations
- `break modify --condition <expr>` — conditional breakpoint
- `break modify --ignore-count <N>` — ignore first N hits
- `break command add [<id>]` — attach actions to a breakpoint
- `tbreak` — create a one-shot temporary breakpoint
- `continue` — resume execution
- `f <N>` — select backtrace frame (`frame select`)
- `help <command>` — inline documentation
- `apropos <keyword>` — search all command descriptions

**Swift 6**
- `@DebugDescription` macro — **[NEW]** custom debugger type summary
- `var debugDescription: String` — string interpolation-based summary property

**Xcode**
- Debug Navigator — backtrace display
- Variable Viewer — structured variable inspection
- QuickLook in variable viewer — rich type preview
- Edit Breakpoint UI — condition, ignore count, action configuration
- Swift Error breakpoint type — **[NEW to spotlight]**
- Control + click "Relaunch" — relaunch without recompile

**dSYM / Symbolication**
- dSYM bundle required for crashlog symbolication (see "Symbolication: Beyond the Basics")

## Code Highlights

`@DebugDescription` macro usage:
```swift
@DebugDescription
struct WatchLaterItem {
    let video: Video
    let name: String
    let addedOn: Date

    var debugDescription: String {
        "\(name) - \(addedOn)"
    }
}
```

Breakpoint action with auto-continue (CLI):
```
b DetailView.swift:70
break command add
p "last video is \(watchLater.last?.name)"
continue
DONE
```

## Takeaways
- Adopt `@DebugDescription` in Swift 6 for any type that appears frequently in the variable viewer or inside collections — it dramatically improves debugger ergonomics.
- Use `p` as the default expression/variable command; reach for `po` or `v` only when you have a specific reason.
- Plant a Swift Error breakpoint when debugging throwing initializers instead of guessing which `try` failed.
- Use breakpoint actions with `continue` to non-invasively log state without recompiling — especially valuable in SwiftUI code where closures fire at unexpected times.

---
_Source: WWDC24 Session 10198 page (abstract, chapter summaries, code samples, and resource links)._
