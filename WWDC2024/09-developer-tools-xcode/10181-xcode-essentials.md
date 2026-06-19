# Xcode Essentials
**WWDC24 · Session 10181** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10181/)

_Platforms:_ macOS (Xcode 16)

## Overview
"Xcode Essentials" is a practical, tips-focused session covering the complete development cycle: edit, debug, commit, repeat. Aimed at developers of any experience level, the session walks through how to find the right code efficiently, move between files quickly, master key commands and shortcuts, leverage git features inside Xcode, debug with breakpoints and the console, write and run tests, and distribute apps via TestFlight and App Store. The session introduces no new APIs but demonstrates a large number of Xcode workflow features — many of which are not discoverable without guidance.

## Key Topics

### Find the Right Content
- **Navigator filtering**: Each navigator has a bottom-bar filter that supports text and specialized tokens (e.g., filtering by target name). The git-status filter shows only files in the upcoming commit, in their full project context.
- **Find Navigator** (Command-Shift-F): Search across the entire project. Narrow results by filtering for additional words on matching lines, or by file name. Hold Command while clicking a disclosure arrow to collapse all siblings — useful for scanning file names before reading results. Select unwanted results and press Delete to hide them from the query (files are not modified). Use the scope menu to search only in selected groups; save custom scopes for reuse. Filter the symbol index (e.g., "Descendant Types") from the "Text" menu above the search field. Use Command-E to put selected text into the find field without disturbing the clipboard.

### Move Between Files
- **Tab bar**: Xcode maintains two tab types — permanent (explicitly opened/edited) and implicit (italicized title; disappears when navigated away). Double-click or "Keep Open" to make a tab permanent. Option-click the close button to close all other tabs. Click-and-hold the back/forward buttons to see a full navigation history.
- **Related files menu**: Shows recent files and symbolic relations (subclasses, callers, etc.) based on cursor position.
- **Jump bar** (below the tab bar): Every path component is interactive — click to browse siblings; start typing to filter the pop-up menu. Press Command-Shift-J to reveal the current file in the project navigator.
- **Creating files**: Right-click anywhere in the navigator for a new empty file (no template chooser). Option-drag to copy a file. Cut code, right-click the navigator while holding Option, and choose "New file with contents of clipboard." Paste content directly into the navigator to create a new file.
- **Warning/error annotations**: Click to expand multi-issue lines; click Fix to apply compiler fix-its. Gray annotations indicate the file has changed since the last build refresh.
- **Bookmarks**: Right-click any line to add a bookmark; manage and check off tasks in the Bookmarks navigator.
- **MARK comments**: `// MARK: Section Title` appears in the mini-map and the editor content jump bar segment.

### Leverage Shortcuts
- **Open Quickly** (Command-Shift-O): Jump to any file or symbol using fuzzy matching. Append `/` to match file paths; append `:N` to jump to a line number. Hold Option when pressing Return to open in a split editor.
- **Quick Actions** (Command-Shift-A): Natural-language search of all Xcode commands.
- Key commands: Command-click → Jump to Definition; Option-click → Show Quick Help (or inferred type for Swift variables); Command-Control-E → Edit All in Scope (rename in file); right-click → Show Callers; Control-M → reformat a call to multiple lines.
- Text movement: Option+arrows → move by word; Control+arrows → move by subword; Command+Left/Right → beginning/end of line. Double-click a bracket/paren/quote to jump to its match.
- **Multi-cursor**: Control-Shift-click to insert multiple cursors; repeat commands in Vim mode act as another form of multi-cursor editing.
- **Code completion in Xcode 16**: Predictive completions suggest whole statements inline; Tab accepts the current suggestion; Option expands the full prediction; Option-Tab accepts it entirely. Comments and expressive variable names improve prediction quality. Hold Option when choosing a completion and press Enter to accept all arguments.
- **Vim mode**: Toggle in Editor menu. Xcode 16 adds support for Vim's repeat command.
- **Emacs commands**: Xcode supports basic Emacs movement shortcuts (Control-A/E/P/N and others) natively.

### Get the Most Out of Git
- **Show Last Change for Line**: Right-click any line → "Show last change for line" — a focused blame overview for that line's commit.
- **Changes Navigator**: Preview all staged and unstaged changes before committing; stage, commit, and review diffs.

### Debugging
- **Basic breakpoints**: Click a line number to set. Click again to disable (without removing). Drag off the gutter to delete.
- **Two-breakpoint technique**: Disable a high-frequency breakpoint; set a trigger breakpoint elsewhere; re-enable on pause and continue — lets you catch a specific invocation of a busy function.
- **Swift Error Breakpoint**: Add from the Breakpoints navigator — stops at the `throw` site, not the `catch` site. Combine with the enabling technique to scope activation.
- **Conditional breakpoints**: Double-click a breakpoint to edit; add a condition expression. The breakpoint only stops when the condition is true.
- **Logging without rebuilding**: Add a debugger expression to a breakpoint (e.g., `p "Username is \(cloudURL.user())"`) and enable "Automatically continue after evaluating actions" — zero-rebuild logging.
- **Retroactive evaluation**: After an unexpected branch, evaluate sub-expressions in the debugger (`p session`, `p cloudURLs.allSatisfy(...)`) to diagnose without restarting the session. Drag the green program counter backward to re-execute code (side effects are not rewound).
- **Disable all breakpoints**: Click the breakpoint button in the debug bar; continue to a clean state; re-enable and re-trigger — useful for repeated debugging runs.
- **Run Without Building** (Command-Control-R): Skip the build step to get back to debugging immediately.
- **Unified Backtrace View** (Xcode 16): Activate from the debug bar — shows a full call stack inline in the editor across all relevant frames.
- **Console / os_log**: Prefer `os_log` over `print` — supports debug levels, source-location jump arrow, metadata filters (type, timestamp, library). Macros (`#fileID`, `#function`) available in print statements.

### Testing
- Run all tests: Command-U. Run a single test: click the diamond next to the function.
- **Re-run last tests**: Command-Control-Option-G — no need to navigate back to the test.
- **Test without building**: Command-Control-U.
- **Test Navigator** (Command-6): Filter to "only included tests" (current plan), filter by text or Swift Testing tag, select a subset and use the context menu to run focused tests. Failures-only filter — tests drop off as fixed.
- **Test statuses**: Green diamond = pass; red X = failure; gray X = expected failure (mark tests as `XCTExpectFailure` / `.disabled`); gray arrow = skipped.
- **Run Test Repeatedly**: Context menu option — run a fixed count or until failure; useful for catching race conditions.
- **xcodebuild test**: `xcodebuild test -scheme MyScheme -testPlan MyPlan` — works with `git bisect` for regression hunting.
- **Xcode Cloud**: 25 compute hours/month free with a developer account; configure workflows triggered on branch push; auto-submit to TestFlight on test pass; automate tester notes from git commit messages.
- **Test Plans**: Create groupings across schemes and targets; each scheme can have multiple plans; edit from Product > Test Plan.
- **Code Coverage**: Enable from the Editor menu; run tests; coverage counts appear in the gutter. Report Navigator (Command-9) shows per-file and per-function coverage.
- **Test Report**: Double-click a failed test to see a side-by-side screen recording and event timeline — pinpoints exactly when and where the failure occurred.

### Distributing Your App
- **TestFlight**: Distribute to up to 10,000 beta testers via email or public link; testers receive automatic updates; feedback and analytics surface in the Xcode Organizer.
- **Archiving**: Product > Archive produces a release build with debug symbols included. Selecting "Distribute App" offers presets: App Store Connect, TestFlight Internal Only, Release Testing, Enterprise, Debugging.
- **Organizer** (Command-Option-Shift-O): Feedback from TestFlight testers; Launches diagnostics (slow launch code-path signatures); Terminations analytics; Disk Writes (with trend arrows in Xcode 16). Privacy-preserving — only users who opted into sharing diagnostics appear.

## APIs & Frameworks

This is a developer-workflow session — no new APIs are introduced. Xcode features referenced:

**Editing**
- Open Quickly (Command-Shift-O)
- Quick Actions (Command-Shift-A)
- Find Navigator filtering with custom scopes
- Command-E → populate find field without clipboard impact
- Option-drag → copy files in navigator
- Multi-cursor (Control-Shift-click)
- Code completion predictive inline suggestions (Xcode 16) **[NEW]**
- Vim mode repeat command (Xcode 16) **[NEW]**
- `#warning("message")` / `// MARK: Title` annotations
- `<#placeholder#>` syntax for code completion templates

**Debugging**
- Breakpoint conditions and logging actions (auto-continue)
- Swift Error Breakpoint in Breakpoints navigator
- `p` / `po` LLDB expressions
- Run Without Building (Command-Control-R)
- Unified Backtrace View (Xcode 16) **[NEW]**
- `os_log` with source-jump arrow in Xcode console

**Testing**
- Test Navigator (Command-6) tag and text filtering
- Re-run last tests (Command-Control-Option-G)
- Test without building (Command-Control-U)
- Run Test Repeatedly (context menu)
- Test Plans (Product > Test Plan)
- Code Coverage (Editor menu toggle + Report Navigator)
- Test Report with screen recording timeline
- `xcodebuild test -scheme … -testPlan …`

**Distribution**
- TestFlight (beta distribution, feedback, analytics)
- Product > Archive + Distribute App presets
- Organizer (Command-Option-Shift-O) — Feedback, Launches, Terminations, Disk Writes

## Code Highlights

`#warning` and `#error` annotations for task management:
```swift
#warning("This is a warning annotation")
#error("This is an error annotation")
```

MARK comment for jump-bar section titles:
```swift
// MARK: This is a section title
```

Code completion placeholder syntax:
```swift
<#placeholder#>
```

Conditional breakpoint expression:
```
cloudURL.scheme == "https"
```

Breakpoint logging action (with "Automatically continue" enabled):
```
p "Username is \(cloudURL.user())"
```

Retroactive guard clause diagnosis in LLDB:
```
p session
p cloudURLs.allSatisfy({ $0.scheme == "https" })
p session.configuration.networkServiceType == .video
```

## Takeaways
- Conditional breakpoints and auto-continuing log actions are zero-rebuild alternatives to `print` statements — set once, log indefinitely, remove without leaving dead code.
- Use the two-breakpoint technique (disable the high-frequency breakpoint, trigger it from a known entry point) to catch a specific invocation of a busy function without pausing on every hit.
- Command-Control-Option-G reruns the last tests from anywhere in Xcode — no need to navigate back to the test function. Command-Control-U runs them without rebuilding.
- The Test Report's screen-recording timeline makes it possible to see exactly when a UI test failure occurred — far more diagnostic than a log line or assertion message alone.

---
_Source: WWDC24 Session 10181 page (abstract, chapter summaries, code samples, and resource links)._
