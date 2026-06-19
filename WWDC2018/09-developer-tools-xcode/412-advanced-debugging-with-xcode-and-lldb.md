# Advanced Debugging with Xcode and LLDB
**WWDC18 · Session 412** · [Watch](https://developer.apple.com/videos/play/wwdc2018/412/)

_Platforms:_ iOS 12, macOS Mojave 10.14

## Overview
This session is a dense, demo-driven tour of advanced Xcode and LLDB debugging techniques. The first half, presented by a frameworks engineer on the Xcode team, covers LLDB expression injection, custom breakpoint configurations, symbolic breakpoints, watchpoints, instruction pointer manipulation, and LLDB Python scripting. The second half covers Xcode's visual view debugger with a focus on Auto Layout constraint debugging and dark-mode appearance debugging new to Xcode 10.

Swift debugging reliability improvements in Xcode 10 are highlighted up front: a new fallback mechanism for failed AST context reconstruction, and many fixes for missing type information and variable values in the debugger.

## Key Topics

### Swift Debugging Reliability (Xcode 10)
- LLDB now falls back to a simplified expression context when the full AST context cannot be reconstructed (module conflicts, complex build configurations).
- Numerous fixes for missing type info and blank variable values in the Variables View.

### LLDB Expression Injection and Breakpoint Configuration
- `expression` / `expr` / `e` — evaluate any Swift or Objective-C expression, modify program state without recompiling.
- Auto-continuing breakpoints with debugger command actions — inject expressions and continue without stopping.
- Breakpoint conditions — expressions evaluated to `Bool` to gate breakpoint triggering.
- Chained breakpoints: use `breakpoint set --one-shot true --name <symbol>` as a debugger command action to create a temporary dependent breakpoint from another breakpoint.
- `thread jump --by 1` — skip the current line without executing it (dangerous; use with care).
- Dragging the instruction pointer (green annotation) to skip or re-execute lines while paused.

### Symbolic Breakpoints
- Created in Xcode's Breakpoint Navigator via the + button.
- Accepts any function/method name in Objective-C selector format (e.g., `-[UILabel setText:]`).
- Shows resolved locations in the navigator; no entries means the symbol was not found.
- Inspect arguments in assembly frames using pseudo-registers `$arg1`, `$arg2`, `$arg3`, etc. (calling-convention-aware).

### Watchpoints
- Created from the Variables View by right-clicking a property and selecting "Watch".
- Pauses the debugger the next time the watched variable is written.
- Visible in the Breakpoint Navigator under a "Watchpoints" group.

### Objective-C Expressions in Swift Frames
- `expression -l objc -O -- <expr>` — evaluate Objective-C in a Swift frame.
- Use backtick substitution inside the expression to reference Swift frame variables: `\`self.view\``.
- `po <addr>` does not work directly for raw addresses in Swift; use `unsafeBitCast(<addr>, to: MyType.self)` instead.
- `command alias poc expression -l objc -O --` — create shorthand LLDB command aliases.

### LLDB Python Scripting
- `command script import <path/to/script.py>` — add custom LLDB commands.
- `lldbinit` file at `~/.lldbinit` — persist aliases and script imports across sessions.
- Python scripts have full access to the LLDB API via the `lldb` module.
- Sample `nudge` command: takes x-offset, y-offset, and view expression; adjusts `center` and flushes `CATransaction`.

### Xcode View Debugger (Xcode 10)
- Capture view hierarchy via touch bar spray-can icon or Command+click on Xcode's debug bar (keeps app in active state).
- "Reveal in Debug Navigator" — locates selected view in the hierarchical outline.
- "Show Clipped Content" — reveals views extending beyond window/parent bounds.
- Constraint inspector — shows `NSLayoutConstraint` properties including first/second items, constant, relation, and multiplier.
- Copy a selected object in the view debugger pastes its memory address into the console — works for all objects in view debugger and memory graph debugger.
- Allocation backtraces visible in the inspector when "Malloc Stack Logging" is enabled in scheme diagnostics.
- `debugDescription` property on any object (or `CustomDebugStringConvertible` conformance) surfaces in the view debugger inspector.
- Dark mode: name colors from asset catalogs and system colors shown in inspector; unnamed RGB colors don't adapt to appearance.
- Appearance override button in Xcode 10 debug bar: switch target app to Light, Dark, High Contrast Light/Dark without changing system appearance.
- Multiple windows appear as multiple root-level items in the view hierarchy outline.
- Command+click in the 3D exploded view to click through front views and select occluded views.

### po / p / frame variable
- `po` — alias for `expression -O`; requests `debugDescription`.
- `p` — alias for `expression`; uses LLDB built-in formatters.
- `frame variable` — reads variable directly from memory, no expression compilation; most reliable fallback when expression evaluation fails.
- Customize `po` output by conforming to `CustomDebugStringConvertible` (Swift) or overriding `debugDescription` (Objective-C).

## APIs & Frameworks

**LLDB**
- `expression` (`expr`, `e`) command
- `po` command (alias for `expression -O`)
- `p` command (alias for `expression`)
- `frame variable` command
- `thread jump --by <N>` command
- `breakpoint set --one-shot true --name <symbol>` command
- `command alias <alias> <command>` command
- `command script import <path>` command
- `$arg1`, `$arg2`, `$arg3` pseudo-registers
- `~/.lldbinit` configuration file

**UIKit (used in debugging targets)**
- `UIView.recursiveDescription()` (private debug method, accessible via `expression -l objc -O`)
- `UILabel.text` (property breakpoint target)
- `UIView.center`
- `UICollisionBehavior`, `UIDynamicAnimator`, `UIDynamicAnimatorDelegate`

**AppKit (macOS debugging targets)**
- `NSLayoutConstraint.constant`
- `NSColor.textColor` (system adaptive color)
- `NSAppearance` — `appearance`, `effectiveAppearance`

**Core Animation**
- `CATransaction.flush()` — force display update while paused in debugger

**Xcode / Developer Tools**
- Xcode Behaviors (Edit > Behaviors) — configure actions on debugger pause, including tab management
- Scheme Diagnostics > Malloc Stack Logging — enables allocation backtraces in view and memory debuggers
- Visual View Debugger — "Show Clipped Content", "Reveal in Debug Navigator", constraint inspector
- Touch Bar debug controls — capture view hierarchy, appearance overrides
- Xcode 10 appearance override in debug bar **[NEW]**

**Swift Standard Library**
- `CustomDebugStringConvertible` protocol — `debugDescription: String`
- `unsafeBitCast(_:to:)` — cast raw address to typed object

## Code Highlights

Auto-continuing breakpoint debugger command to inject expression:
```
expression didReachSelectedHeight = false
```

One-shot breakpoint set as a debugger command action:
```
breakpoint set --one-shot true --name "-[UILabel setText:]"
```

Skip line and substitute alternative call:
```
thread jump --by 1
expression jumpAstronaut(animated: false)
```

Evaluate Objective-C in a Swift frame and print view hierarchy:
```
expression -l objc -O -- [`self.view` recursiveDescription]
```

Custom LLDB alias for printing objects by address (Objective-C):
```
command alias poc expression -l objc -O --
```

Force Core Animation to update display while paused:
```
expression CATransaction.flush()
```

Python script skeleton for custom LLDB command (`~/.lldbinit`):
```
command script import ~/lldb/nudge.py
```

Swift `CustomDebugStringConvertible` conformance:
```swift
extension GamePlay: CustomDebugStringConvertible {
    var debugDescription: String {
        return "GamePlay(attempts: \(attempts), maxAttempts: \(maxAttempts), score: \(score))"
    }
}
```

## Takeaways
- Auto-continuing breakpoints with `expression` actions let you test code changes live without recompile/rerun cycles — essential for hard-to-reproduce bugs.
- Combine chained breakpoints (one-shot), conditions, and `thread jump` to surgically control execution flow during a debugging session.
- LLDB Python scripting and `~/.lldbinit` aliases dramatically reduce repetitive console typing for common debugging patterns.
- Xcode 10's view debugger constraint inspector, clipped-content view, and allocation backtraces together make Auto Layout bugs tractable; the appearance override in the debug bar makes dark-mode testing seamless.

---
_Source: WWDC18 Session 412 page (abstract, full transcript, and resource links)._
