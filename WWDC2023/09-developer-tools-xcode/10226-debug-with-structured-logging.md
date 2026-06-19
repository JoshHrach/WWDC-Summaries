# Debug with Structured Logging
**WWDC23 · Session 10226** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10226/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10, tvOS 17, visionOS 1

## Overview
Xcode 15 ships a completely redesigned Debug Console that makes working with Apple's Unified Logging system (`OSLog`) dramatically more efficient. The session covers four areas: a tour of the new console's metadata display, metadata filtering, and log visualization; a live debugging walkthrough using category-based filtering to locate a real bug; the new `dwim-print` (Do What I Mean Print) LLDB command that replaces both `p` and `po` with a smarter single interface; and best-practice guidance for migrating from `print()` to `OSLog` and getting the most from structured logging.

## Key Topics

### Redesigned Debug Console (Xcode 15)

**Cleaner output**: The console no longer prefixes every log line with timestamp, subsystem, category, and other metadata. The developer-intended message is front and center.

**Metadata on demand**: The Metadata Options button (lower-left) lets you toggle which metadata fields appear—Type (debug/info/error/fault), Library, Subsystem, Category. When enabled, metadata renders below each log in a smaller, subtler style.

**Severity coloring**: Error logs have a yellow background; fault logs have a red background, making high-priority items immediately visible.

**Quick Look**: Select any log and press Space to show a popover with all available metadata including call site (function name, file, line).

**Tokenized filtering**: The filter bar supports complex, token-based filters with autocomplete. Filters can be entered manually, selected from the filter menu (quick-access for log type filters), or created via secondary-click on any log → "Show only logs like this" / "Hide logs like this." This makes it practical to narrow a dense console to only the relevant subsystem or category during investigation.

### Live Debugging Walkthrough
The session demonstrates diagnosing a data-not-saved bug in the Backyard Birds app:
1. Reproduce the issue (display name change not persisting).
2. Filter the console to `category: account` using the tokenized filter, reducing output to account-related logs only.
3. Hover a relevant log → click the source location indicator → Xcode jumps to the emitting code.
4. Trace through `setDisplayName(_:)` to find that the database was being updated but the local cache (`account.displayName`) was not.
5. Add the missing cache assignment; set a breakpoint to verify with `p account`.

This workflow — write expressive `OSLog` statements in code, then use the console's filter system to slice through output by subsystem/category at debug time — is the model the session advocates.

### LLDB: dwim-print (Do What I Mean Print)
Xcode 15 introduces `dwim-print`, a new LLDB command that chooses the fastest and most appropriate evaluation strategy automatically:
- If the expression is a simple variable reference, it evaluates like `frame variable` (fast, no compile step).
- If the expression needs evaluation (method calls, computed properties), it compiles and runs the expression like `expression`.
- Falls back gracefully, never just printing a memory address when a richer result is available.

**Alias changes**:
- `p` now aliases `dwim-print` (was `expression`).
- `po` now aliases `dwim-print --object-description` (prints `CustomStringConvertible` / `debugDescription` if available).

This means the old habit of trying `po`, getting a raw address, then switching to `p` is no longer necessary—just use `p` for variable inspection and `po` only when you specifically want the custom object description.

### Tips for Logging: Migrating from print() to OSLog
`print()` is for command-line UI output. For diagnostics, always use `OSLog`:

**Create a `Logger` per component**:
```swift
import OSLog
let logger = Logger(subsystem: "com.example.BackyardBirds", category: "Account")
```
Use bundle identifier as subsystem, class/component name as category.

**Use appropriate log levels**:
- `.debug` — verbose developer-only information (redacted in release builds)
- `.info` — routine operational events
- `.notice` — significant events worth noting
- `.error` — recoverable errors
- `.fault` — unrecoverable programmer errors

**Benefits over print()**:
- Metadata (timestamp, process ID, thread ID, source location) captured automatically.
- Logs are filterable in the console by subsystem and category.
- Available in `OSLogStore` for field diagnostics from user-reported issues.
- Works as a tracing facility with Instruments (Logging template, signposts).

**Get the most from logging**:
- Create separate `Logger` instances for distinct app components—better filterability.
- Use `OSLogStore` to collect diagnostics after field issues; accessible from the host Mac or via crash reports.
- Use signposts (`OSSignposter`) alongside logs for performance analysis in Instruments.

## APIs & Frameworks

**OSLog (existing, best practices reinforced)**
- `Logger(subsystem:category:)` — per-component log handle
- `Logger.debug(_:)`, `.info(_:)`, `.notice(_:)`, `.error(_:)`, `.fault(_:)` — leveled logging
- `OSLogStore` — access stored logs programmatically for field diagnostics
- String interpolation with privacy annotation: `"\(value, privacy: .public)"` — control log redaction

**Xcode 15 Debug Console — New Features**
- Metadata Options toggle **[NEW]** — per-session metadata display control (Type, Library, Subsystem, Category)
- Tokenized filter bar **[NEW]** — autocomplete-assisted, token-based log filtering
- Filter menu **[NEW]** — quick-access filters by log type
- Secondary-click filter shortcuts **[NEW]** — "Show only" / "Hide" similar logs
- Log quick look (Space key) **[NEW]** — full metadata popover with call site
- Source location jump **[NEW]** — hover log → click source location → jump to emitting code

**LLDB (Xcode 15)**
- `dwim-print` **[NEW]** — smart print command choosing fastest evaluation strategy
- `p` — now aliases `dwim-print` (was `expression`)
- `po` — now aliases `dwim-print --object-description`

## Code Highlights

Proper OSLog setup:
```swift
import OSLog

let logger = Logger(subsystem: "BackyardBirdsData", category: "Account")

func login(password: String) -> Error? {
    var error: Error? = nil
    logger.info("Logging in user '\(username)'...")

    // ... authentication logic ...

    if let error {
        logger.error("User '\(username)' failed to log in. Error: \(error)")
    } else {
        loggedIn = true
        logger.notice("User '\(username)' logged in successfully.")
    }
    return error
}
```

LLDB variable inspection (before and after):
```
# Old workflow — unreliable
(lldb) po account
<Account: 0x60000223b2a0>   # raw address when no CustomStringConvertible

(lldb) p account            # had to switch commands

# New workflow — use p for everything
(lldb) p account
(BackyardBirdsData.Account) {
    id = 3A9FC684-8DFC-4D7D-B645-E393AEBA14EE
    displayName = "Johnny Appleseed"
    emailAddress = "johnny_appleseed@icloud.com"
    isPremiumMember = true
}
```

## Resources
- [Logging documentation](https://developer.apple.com/documentation/os/logging)
- Related: "What's new in Xcode 15" (WWDC23 10165)
- Related: "Explore logging in Swift" (WWDC20 10168)
- Related: "Measuring Performance Using Logging" (WWDC18 405)

## Takeaways
- The Xcode 15 Debug Console's tokenized filter bar — especially filtering by category — is the fastest way to narrow a noisy console to the relevant code path; invest in meaningful `subsystem` and `category` names.
- Replace `po` with `p` as the default inspection command; `dwim-print` chooses the right evaluation strategy automatically, and `po` is now reserved for when you explicitly need `CustomStringConvertible` output.
- `print()` should essentially never be used for diagnostic logging in production code; `OSLog` provides automatic metadata, privacy controls, Instruments integration, and `OSLogStore` access at no extra cost.
- Severity levels (debug/info/notice/error/fault) map directly to the console's color coding, making logs immediately scannable for critical issues.

---
_Source: WWDC23 Session 10226 page (abstract, transcript, chapter summaries, code samples, and resource links)._
