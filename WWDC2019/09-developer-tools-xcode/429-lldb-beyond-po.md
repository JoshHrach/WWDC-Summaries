# LLDB: Beyond "po"
**WWDC19 · Session 429** · [Watch](https://developer.apple.com/videos/play/wwdc2019/429/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
This session takes an in-depth look at the three main LLDB commands for printing values during debugging — `po`, `p`, and `v` — explaining what each one does internally, where their trade-offs lie, and when to use each. Understanding the compile-and-execute pipeline behind `po` and `p` vs. the memory-reading approach of `v` helps developers choose the right tool and avoid confusing errors.

The second half covers LLDB's fully extensible data formatter subsystem: filters, string summaries, and synthetic children — all definable either directly in the LLDB console or in external Python 3 scripts that leverage LLDB's scripting bridge (`SBValue`, `SBFrame`, `SBTarget`, etc.). Xcode 11 migrates LLDB scripting from Python 2 to Python 3, and the session provides migration guidance.

## Key Topics
- **`po` (print object description)** — compiles and executes an expression, then compiles and executes a second snippet to retrieve the object's `debugDescription`; can evaluate arbitrary expressions
- **`p` (expression without description)** — compiles and evaluates an expression, performs dynamic type resolution once on the result, then passes it to the data formatter subsystem; result is given an incrementing name (`$R0`, `$R1`, …) reusable in later expressions
- **`v` (frame variable)** — does NOT compile code; reads the variable directly from memory, performs dynamic type resolution recursively at each subfield access; fastest option; does not support computed properties or overloaded operators
- **Dynamic type resolution** — LLDB inspects runtime metadata to display the most accurate concrete type rather than the declared static type
- **Data formatters** — filters (hide irrelevant fields), summaries (single-line string description), synthetic children (custom expansion in variables view); all three affect both LLDB console and Xcode Variables View
- **Python 3 scripting** — use `command script import`, `type summary add`, `type synthetic add`; `SBValue`, `SBFrame`, `SBTarget`, `SBProcess`, `SBThread` objects; Xcode 11 requires Python 3
- **`.lldbinit`** — persist custom formatters and aliases across debug sessions

## APIs & Frameworks
- **LLDB**
  - `po` — alias for `expression --object-description --`
  - `p` — alias for `expression`
  - `v` — alias for `frame variable` **[introduced Xcode 10.2]**
  - `expression` — full expression evaluator
  - `frame variable` — memory-based variable inspector
  - `command alias` — define custom LLDB command aliases
  - `type summary add` — register a string or Python summary formatter for a type
  - `type synthetic add` — register a Python synthetic children provider for a type
  - `type filter add` — restrict which children are shown for a type
  - `command script import` — load a Python script file into LLDB
  - `script` — drop into interactive Python REPL
  - `--raw` flag on `p`/`v` — bypass formatters to see raw representation
- **LLDB Python Scripting Bridge** (Python 3, Xcode 11+)
  - `lldb.frame` — current `SBFrame`
  - `SBFrame.FindVariable(name)` — locate a variable by name
  - `SBValue.GetChildMemberWithName(name)` — access a named child field
  - `SBValue.GetChildAtIndex(index)` — access an indexed child
  - `SBValue.GetNumChildren()` — number of children
  - `SBValue.GetSummary()` — formatted summary string
  - `SBValue.description` — data formatter output
  - `SBTarget`, `SBProcess`, `SBThread` — runtime context objects
- **Swift / Objective-C protocols**
  - `CustomDebugStringConvertible` — `.debugDescription` used by `po`
  - `CustomReflectable` — customizes substructure visible to `po`
  - `-[NSObject description]` — Objective-C equivalent for `po`

## Code Highlights

```swift
// Conform to CustomDebugStringConvertible for custom po output
struct Trip: CustomDebugStringConvertible {
    var name: String
    var destinations: [String]
    var debugDescription: String {
        return "Trip '\(name)' visiting \(destinations.joined(separator: ", "))"
    }
}
```

```
# LLDB console — print commands
(lldb) po cruise
(lldb) p cruise
(lldb) v cruise
(lldb) v cruise.destinations[0]  # dynamic type resolution at each step
```

```python
# Python 3 summary provider (Trip.py)
import lldb

def trip_summary(value, internal_dict):
    destinations = value.GetChildMemberWithName("destinations")
    count = destinations.GetNumChildren()
    if count == 0:
        return "Empty trip"
    first = destinations.GetChildAtIndex(0).GetSummary()
    last  = destinations.GetChildAtIndex(count - 1).GetSummary()
    name  = value.GetChildMemberWithName("name").GetSummary()
    return f"{name}: {first} → {last}"
```

```
# Load and register in LLDB
(lldb) command script import ~/Trip.py
(lldb) type summary add -F Trip.trip_summary Trip
```

## Takeaways
- Use `v` by default for fast, recursively-resolved variable inspection; use `p` when you need expression evaluation; use `po` when you need the object description (custom `debugDescription`).
- The `p` command binds results to `$R0`, `$R1`, … which can be referenced in subsequent expressions.
- Data formatters (filters, summaries, synthetic children) affect both the LLDB console and Xcode's Variables View, making them a powerful debugging productivity investment.
- LLDB scripting moved to Python 3 in Xcode 11; existing Python 2 scripts need migration. Persist formatters in `~/.lldbinit`.

---
_Source: WWDC19 Session 429 page (abstract, chapter summaries, code samples, and resource links)._
