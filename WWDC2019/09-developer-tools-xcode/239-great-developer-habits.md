# Great Developer Habits
**WWDC19 · Session 239** · [Watch](https://developer.apple.com/videos/play/wwdc2019/239/)

_Platforms:_ iOS, iPadOS, macOS, watchOS, tvOS (all platforms / general practices)

## Overview
A non-API session from Apple's Technology Evangelism team that catalogs the professional development habits shared by the most successful app developers Apple works with. The session organizes practices into seven areas: organization, tracking (source control), documentation, testing, analysis, evaluation (code review), and dependencies.

Each area is presented as a set of concrete, actionable habits — things that feel optional individually but collectively determine whether a project stays maintainable and reliable over its lifetime. The session emphasizes that these practices have compounding value: investing small amounts of time consistently prevents the exponential cost of catching the same problems much later.

## Key Topics

**Organization**
- Use Xcode groups that mirror the filesystem structure (enforced since Xcode 9 with folder creation on disk)
- Organize groups functionally (how users interact with the app) rather than by file type
- Use Storyboard References to split large storyboards — one per major section; avoids merge conflicts and isolates changes
- Keep the Xcode project file modern: accept format upgrades when prompted; use the new build system (default since Xcode 10; required for Swift Packages)
- Delete unused and commented-out code; history lives in source control
- Maintain a zero-warning policy; treat warnings as errors and fix them immediately

**Source Control**
- Enable Git at project creation (checkbox in New Project dialog)
- Keep commits small, localized, and self-contained
- Write meaningful commit messages — "notes to future self"
- Use branches for features and bug fixes; squash before merging

**Documentation and Comments**
- Comments explain *why*, not *what* — well-written code is self-documenting algorithmically; comments provide the rationale and context
- Use descriptive variable/constant names instead of abbreviations (`id`, `idx`, single letters)
- Generate doc comment stubs: cursor on function signature → Option+Cmd+/ — fills in `/// - Parameter:` and `/// - Returns:` placeholders
- Option+click any call site to see your own documentation in Quick Help

**Testing**
- Write unit tests as part of implementing each feature, not afterward
- Run tests before every commit
- Unit tests are the key ingredient for continuous integration
- Even "simple" code can regress; automated tests catch regressions before they reach the UI

**Analysis Tools (Run Regularly)**
- Network Link Conditioner: simulate cellular/poor network conditions to surface loading/race conditions
- Address Sanitizer: memory corruption, buffer overflows (common source of security vulnerabilities)
- Thread Sanitizer: data races (Simulator-only; unavailable on device)
- Undefined Behavior Sanitizer: division by zero, out-of-range casts, pointer misalignment
- Main Thread Checker: invalid AppKit/UIKit usage on background threads; minimal overhead, leave enabled
- Debug Gauges (Debug Navigator): CPU, memory, disk, network utilization while running
- Instruments / Time Profiler: deep-dive into performance hotspots; identify work to make async

**Code Review**
- No code merges without review — Apple policy on all internal teams
- Read every changed line; actually build and run the project; run tests
- Review comments and documentation for completeness and spelling
- Check variable name spelling for consistency (affects searchability)
- Code review accelerates skill development and spreads knowledge of the codebase across the team
- Solo developers: establish peer review exchanges with other developers via meetups, conferences, co-working spaces

**Packages, Frameworks, and Dependencies**
- Extract shared code into frameworks to reduce binary size (main app and extensions share one copy)
- Swift Packages in Xcode 11 enable community sharing with tight IDE integration
- Framework code requires especially thorough documentation
- Before adding any external dependency: understand what data it accesses and where it sends it; check transitive dependencies; have a plan if the dependency breaks or disappears

## APIs & Frameworks

_This session is primarily about practices rather than specific APIs. Tools and features referenced:_

**Xcode**
- New build system (default since Xcode 10) — required for Swift Package support
- Storyboard References — embed references between storyboard files
- Source Control integration — Git; commit dialog, branch visualization
- Option+Cmd+/ — generate doc comment stub for a function/property/type
- Debug Gauges — CPU, Memory, Disk, Network panes in Debug Navigator
- Scheme diagnostics: Address Sanitizer, Thread Sanitizer, Undefined Behavior Sanitizer, Main Thread Checker (all toggleable in scheme settings)

**Network Link Conditioner**
- macOS System Preferences / iOS Settings pane for simulating constrained network environments

**Instruments**
- Time Profiler — identify CPU hotspots; surface work that should be async
- Launch from Debug Gauges "Profile in Instruments" button or Xcode Product menu

**XCTest**
- `XCTestCase` — base class for unit tests
- Run before each commit; integrate with CI

## Code Highlights

Generating a doc comment (Option+Cmd+/ on function signature):
```swift
/// Fetches the session list for the specified conference year.
///
/// - Note: This method executes asynchronously.
/// - Parameter year: The four-digit conference year (e.g., 2019).
/// - Parameter completion: Called with the session list on success, or nil on failure.
func fetchSessions(for year: Int, completion: @escaping ([Session]?) -> Void) { ... }
```

Descriptive naming instead of abbreviations:
```swift
// Poor
let id = "A1B2C3"

// Better
let conferenceRegistrationID = "A1B2C3" // Generated by registration system at check-in
```

## Takeaways
- A zero-warning policy and functional group organization are the two fastest wins for a codebase's long-term health — both are nearly free to adopt from day one.
- Leave Address Sanitizer, Thread Sanitizer, and Main Thread Checker enabled in debug schemes at all times; the overhead is minimal and the bugs they surface are expensive to debug manually.
- Write unit tests at implementation time, not afterward; the discipline of writing them before committing is more important than writing comprehensive tests later.
- Every external dependency is a liability as well as an asset — vet what data it touches, what it depends on, and have a replacement plan before adopting it.

---
_Source: WWDC19 Session 239 page (abstract, chapter summaries, code samples, and resource links)._
