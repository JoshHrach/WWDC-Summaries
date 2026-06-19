# Modeling in Custom Instruments
**WWDC19 · Session 421** · [Watch](https://developer.apple.com/videos/play/wwdc2019/421/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
This session is a deep dive into the modeler layer of the Custom Instruments architecture introduced at WWDC18. A modeler is a CLIPS-based rules engine that sits between raw OS Signpost events and the displayable output tables of a custom Instruments instrument. The session explains when to write a custom modeler (versus using Xcode's auto-generated one), how to design the three-step process (define output → instrument with Signposts → write rules), and how to understand and optimize rule execution.

The concrete example throughout is a "mobile agent" pattern — a task decomposition model similar to futures/promises — traced through an iOS app that sorts a list. Signposts are emitted at activity-boundary events (rather than as begin/end pairs), and the modeler uses CLIPS facts and rules to open and close intervals, maintain working memory, and write to output tables. Advanced topics include logical loops and how to avoid them, rule salience and CLIPS modules for controlling execution order, and "speculation mode" for rendering open intervals in real time during live recording.

## Key Topics
- **Custom modeler architecture** — modeler receives time-ordered OS Signpost facts from analysis core, maintains a working memory of "facts", runs a rules engine (CLIPS), and writes to bound output tables
- **When to write a custom modeler** — fuse multiple input tables; maintain running totals or open interval tracking in working memory; synthesize custom graph data (running averages, Kalman filters); generate smarter instruments that understand app semantics
- **Three-step process** — (1) decide what to model / define output table columns; (2) instrument app with `os_signpost` calls; (3) write and iterate on CLIPS rules
- **CLIPS rules engine** — lefthand side (declarative pattern matching against working memory facts) and righthand side (imperative: `assert`, `retract`, `modify`, output table write functions)
- **Fact lifecycle** — `assert` adds a fact with a unique address (`f-N`); `retract` removes it; `modify` = retract + reassert (triggers rules re-firing)
- **Logical loops** — accidental infinite loops from `modify` re-activating the same rule; fixed by goal-oriented programming (assert a goal fact, retract it when satisfied)
- **Rule salience** — integer priority controlling agenda ordering; use sparingly; CLIPS modules (`modeler::`, `recorder::`) provide cleaner execution ordering — modeler module fully drains before recorder module runs
- **CLIPS `focus`** — inline focus shift to a custom lookup module to run a block of rules immediately after an assertion, before returning to the modeler agenda
- **Speculation mode** — a `Speculate` fact is injected into working memory when the modeler needs to write placeholder rows for open intervals up to the event horizon; used for live UI update during recording and for recording final state at trace stop
- **Debugging & profiling** — `log-narrative` function for Printf-style logging viewable in Instruments Inspector modeler log table; profiling mode shows rule activation counts and time distribution per rule

## APIs & Frameworks
- **Instruments / Custom Instruments package (.instrpkg)**
  - Custom modeler target in Xcode
  - `modeler::` module prefix — pure reasoning rules
  - `recorder::` module prefix — output-writing rules
  - Custom CLIPS module definition and `focus` command
- **CLIPS language** — open-source rules engine (circa 1985); used for all Instruments modeler logic
  - `defrule` — define a rule with LHS pattern and RHS actions
  - `assert` — add a fact to working memory
  - `retract ?fact` — remove a fact
  - `modify ?fact (slot value)` — update a fact slot (retract + reassert)
  - `not` — match absence of a pattern in working memory
  - `declare (salience N)` — set rule priority (default 0)
  - `defmodule` — define a named agenda (module)
  - `focus module-name` — run all activations in named module before returning
  - `log-narrative` — Printf-style logging to Instruments Inspector modeler log
  - Profile mode — rule activation counts, time distribution
  - `Speculate` fact — injected by analysis core to trigger speculation output
- **OS Signpost API** (app instrumentation side)
  - `os_signpost(.event, log:, name:, signpostID:, "%{public}s", message)` — event-style Signpost at activity boundaries (50% fewer Signposts vs. begin/end)
  - `os_signpost(.begin, ...)` / `os_signpost(.end, ...)` — interval-style (alternative)
  - Signpost maps to CLIPS fact slots: `name`, `event-type`, `identifier`, `message`
  - Use integer type codes instead of strings in Signpost messages to reduce trace buffer overhead
- **Instruments Inspector**
  - Modeler log table — shows `log-narrative` output
  - Modeler tab — shows rule profiling data (activation count, % time per rule)
  - Log mode selector: none / narrative / profile 1-3

## Code Highlights

```clips
; Detect a mobile agent from an OS Signpost fact
(defrule modeler::detect-mobile-agent
  (os-signpost (name "mobile agent moved")
               (identifier ?id)
               (message ?msg))
  (not (mobile-agent (id ?id)))  ; don't create duplicates
  =>
  (assert (mobile-agent (id ?id) (kind sentinel))))

; Lookup module: resolve agent kind code to string
(defrule lookup::resolve-agent-kind
  ?agent <- (mobile-agent (id ?id) (kind sentinel))
  (type-string-map (code ?code) (string ?str))
  =>
  (modify ?agent (kind ?str)))

; When detecting agent, immediately run the lookup module
(defrule modeler::process-agent
  ...
  =>
  (assert (mobile-agent (id ?id) (kind sentinel)))
  (focus lookup))  ; drain lookup agenda before returning

; Speculation: write open intervals during live recording
(defrule recorder::speculate-open-interval
  (Speculate (event-horizon ?end))
  ?interval <- (open-interval (start ?start) (agent ?agent))
  =>
  (bind ?duration (- ?end ?start))
  ; write placeholder row to output table
  (create-row ... duration: ?duration ...))
```

```swift
// OS Signpost instrumentation in the app
import os.signpost
let log = OSLog(subsystem: "com.example.GoatList", category: "MobileAgent")

func executeStop() {
    os_signpost(.event, log: log, name: "mobile agent executes",
                signpostID: agentID, "%{public}d", agentTypeCode)
    // ... perform stop logic
}

func visitNextStop() {
    os_signpost(.event, log: log, name: "mobile agent moves",
                signpostID: agentID, "%{public}d %{public}d", fromStop, toStop)
    // ... perform movement logic
}
```

## Takeaways
- Custom modelers are the only way to fuse multiple input tables, maintain running state, and synthesize computed data in a Custom Instruments instrument — auto-generated modelers are just a quick-start aid.
- Use event-style Signposts at activity boundaries (instead of begin/end pairs) to halve trace buffer usage; encode type information as integers rather than strings for further savings.
- Avoid logical loops by goal-oriented fact management: assert a goal, fire a separate rule to consume it, so `modify` doesn't re-activate the original rule.
- Use CLIPS modules (`modeler::` vs `recorder::`) to separate reasoning from output writing; use `focus` to run a lookup block immediately after an assertion without fighting salience.
- Implement speculation-mode recorder rules so open intervals render live during recording and are properly committed at trace end.

---
_Source: WWDC19 Session 421 page (abstract, chapter summaries, code samples, and resource links)._
