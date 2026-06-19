# What's New in LLDB
**WWDC15 · Session 402** · [Watch](https://developer.apple.com/videos/play/wwdc2015/402/)

_Platforms:_ iOS 9, OS X El Capitan 10.11, watchOS 2

## Overview
This session by Kate Stone, Sean Callanan, and Enrico Granata covers the major LLDB improvements shipping with Xcode 7, focused on three areas: expression evaluation for Objective-C (full SDK module import, NSLog, inline functions, and constants now work without workarounds), Swift 2 debugging (error handling breakpoints, type-specific error breakpoints, Swift REPL error catching), and a brand-new in-process formatting system for Swift types.

The session also reviews changes since WWDC14 — named/tagged breakpoints settable in `.lldbinit`, Swift type improvements (inherited ObjC fields, command aliases in help), improved data formatting for `Set` and `NSIndexPath`, fixed `printf` variadic prototype, and significantly smaller debug information in Xcode 7 (up to 6x reduction for C++ projects via module-based debug info deduplication).

The new Swift in-process formatting system (`CustomStringConvertible`, `CustomDebugStringConvertible`, `CustomPlaygroundQuickLookable`, `CustomReflectable`) makes custom type representations work identically in the REPL, playgrounds, and the LLDB `po` command — with no Python scripts required.

## Key Topics

### Named / Tagged Breakpoints **[NEW in Xcode 7 / shipped spring 2015]**
- `breakpoint set -n <name>` assigns a name/tag to a breakpoint; names need not be unique; one breakpoint can have multiple names.
- All breakpoint commands (`enable`, `disable`, `delete`) accept names, operating on all breakpoints sharing that name.
- Breakpoints set in `~/.lldbinit` are inherited by every subsequent debug target, enabling a persistent personal breakpoint library.
- Example use: set breakpoints on `malloc`/`free`, tag both as `memory`, then `breakpoint disable memory` by default and enable when needed.

### Objective-C SDK Module Import **[NEW]**
- Previously, LLDB's ObjC expression parser only saw symbols in debug information — not full SDK type information. `NSLog` had unknown return type; `NSMakeRect` (inline function) did not exist; `NSApplication.sharedApplication.undoManager` returned `id` instead of a typed pointer.
- New command: `expression @import AppKit` (or `@import UIKit`) — imports the full SDK module into the LLDB expression parser for the session.
- After import: `NSLog` works correctly, `NS_INLINE` functions like `NSMakeRect` resolve, and typed return values (including nullability) are visible from SDK headers.
- Also fixes: SDK constants (`NSASCIIStringEncoding`), macros (`INT_MAX`, `MAX()`), and removes the need for manual casts.

### Swift 2 Error Handling in LLDB **[NEW]**
- Swift `throws` is supported in LLDB expressions without explicit `try`; LLDB catches and returns errors as an `error` variable automatically.
- **Breakpoint on all Swift errors**: `breakpoint set -E swift` — stops whenever any Swift error would be thrown.
- **Breakpoint on Objective-C exceptions**: `breakpoint set -E objc` — existing behavior, still supported.
- **Type-specific error breakpoint**: `breakpoint set -E swift -O <ErrorTypeName>` — stops only when an error of a specific Swift type would be thrown. **[NEW]**
- REPL supports `do-catch` patterns for NSError just as in normal Swift code.

### Swift In-Process Formatting (CustomStringConvertible etc.) **[NEW]**
- Four Swift protocols power custom type display in LLDB `po`, REPL, and Playgrounds — all using the same mechanism as Swift Playgrounds (now public API in Xcode 7).
- `CustomStringConvertible` — `var description: String`; used by `print()`, string interpolation, and LLDB `po`.
- `CustomDebugStringConvertible` — `var debugDescription: String`; used by `debugPrint()` and LLDB `po` (preferred when both conformances exist).
- `CustomPlaygroundQuickLookable` — `var customPlaygroundQuickLook: PlaygroundQuickLook`; provides rich graphical representation in Playground sidebars. **[NEW public API]**
- `CustomReflectable` — `var customMirror: Mirror`; defines a completely custom children hierarchy for the type, controlling how LLDB and Playgrounds decompose nested objects. **[NEW]**
- Conformances can be added at runtime via the LLDB expression parser (`expression extension MyType: CustomStringConvertible { ... }`) — useful when debugging a hard-to-reproduce bug without restarting.
- In-process formatters run app code — must not mutate the object being described.

### Type Lookup Command **[NEW]**
- `type lookup <TypeName>` (abbreviated `ty l`) — prints a header-like summary of any type visible to the debugger, including Swift protocols, struct fields, and method signatures.
- Works for Swift standard library types (`Error`), custom app types, and ObjC classes.

### Debug Information Size Reduction **[NEW]**
- Xcode 7 builds module-based debug information: debug info for a framework is compiled once and not duplicated in every `.o` file.
- C++ one-definition-rule deduplication further reduces redundancy.
- Result: debug info up to 6x smaller for large C++ projects compared to Xcode 6.
- Final `.dSYM` still contains everything needed for symbolication.

## APIs & Frameworks

- `LLDB` — debugger framework embedded in Xcode
- `breakpoint set -n <name>` **[NEW]** — named/tagged breakpoints
- `breakpoint enable/disable/delete <name>` **[NEW]** — operate on breakpoints by name/tag
- `~/.lldbinit` — LLDB session initialization file; breakpoints set here are inherited by all targets
- `expression @import <Module>` **[NEW]** — import SDK modules into the LLDB expression parser for ObjC sessions
- `breakpoint set -E swift` **[NEW]** — catch all Swift errors as breakpoints
- `breakpoint set -E swift -O <ErrorTypeName>` **[NEW]** — catch specific Swift error type
- `breakpoint set -E objc` — catch Objective-C exceptions
- `type lookup <TypeName>` **[NEW]** — print type definition summary from debugger
- `frame variable` / `frv` — inspect local variables without executing code
- `expression` / `p` — compile and execute an expression in the target process
- `expression -O --` / `po` — execute expression and call `description`/`debugDescription`
- `CustomStringConvertible` protocol (`var description: String`) **[NEW public in Xcode 7]**
- `CustomDebugStringConvertible` protocol (`var debugDescription: String`) **[NEW public in Xcode 7]**
- `CustomPlaygroundQuickLookable` protocol (`var customPlaygroundQuickLook: PlaygroundQuickLook`) **[NEW]**
- `CustomReflectable` protocol (`var customMirror: Mirror`) **[NEW]**
- `Mirror` struct — reflects the children/structure of a Swift value **[NEW]**
- `PlaygroundQuickLook` enum — rich graphical data types for Playground sidebar **[NEW]**
- Swift REPL — LLDB-backed interactive Swift environment; supports breakpoints and `do-catch`

## Code Highlights

Import SDK into LLDB expression parser (ObjC):
```
(lldb) expression @import AppKit
(lldb) po NSApplication.sharedApplication().undoManager
```

Named breakpoints in `~/.lldbinit`:
```
breakpoint set -n malloc -N memory
breakpoint set -n free -N memory
breakpoint disable memory
```

Catch a specific Swift error type:
```
(lldb) breakpoint set -E swift -O NetworkError
```

Custom string representation for `po`:
```swift
extension TemperatureData: CustomDebugStringConvertible {
    var debugDescription: String {
        let formatted = DateFormatter.localizedString(from: time, dateStyle: .none, timeStyle: .short)
        return "\(formatted): \(celsius)°C / \(fahrenheit)°F"
    }
}
```

Custom Mirror for nested structure display:
```swift
extension TemperatureData: CustomReflectable {
    var customMirror: Mirror {
        return Mirror(self, children: [
            "time": DateFormatter.localizedString(from: time, dateStyle: .none, timeStyle: .short),
            "celsius": celsius,
            "fahrenheit": celsius * 9/5 + 32
        ])
    }
}
```

## Takeaways
- `expression @import AppKit` (or UIKit) is the single command that unlocks full SDK type information in LLDB's ObjC expression parser — fixes NSLog, inline functions, constants, and typed return values.
- Named breakpoints in `~/.lldbinit` create a persistent, reusable breakpoint library that applies to every debug session automatically.
- The four `Custom*` Swift protocols (String/DebugString/QuickLook/Reflectable) provide a unified, code-level formatting system that works in LLDB `po`, the Swift REPL, and Playgrounds without any Python scripting.
- Type-specific Swift error breakpoints (`-E swift -O <Type>`) let you zero in on exactly which error type is causing a `throws` to fire without stopping at every throw in the program.

---
_Source: WWDC15 Session 402 page (abstract, chapter summaries, code samples, and resource links)._
