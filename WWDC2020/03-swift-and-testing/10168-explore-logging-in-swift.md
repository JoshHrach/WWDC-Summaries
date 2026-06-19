# Explore Logging in Swift
**WWDC20 · Session 10168** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10168/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
This session introduces the new Swift unified logging APIs available in Xcode 12 and iOS 14, built on top of the existing `os_log` system. The new `Logger` type provides first-class Swift string interpolation for log messages, enabling both strong privacy defaults and rich formatting options — all while maintaining high performance through compiler-level optimizations that avoid unnecessary string allocation.

The presentation uses a real-world "Fruta" smoothie app scenario to demonstrate how logs can help diagnose hard-to-reproduce bugs — specifically a race condition involving network tasks and user interaction — without needing to reproduce the problem under a debugger. The workflow covers adding logging, collecting archives with `log collect`, and filtering in Console app.

A second demo shows how formatting options such as fixed-width alignment and floating-point precision can turn raw log output into structured, column-selectable data that can be pasted directly into Numbers for analysis.

## Key Topics

**Three-Step Logging Setup**
- Import the `os` module
- Create a `Logger` with subsystem (bundle ID) and category
- Call logging methods using Swift string interpolation

**Privacy Controls**
- Non-numeric runtime values are redacted as `<private>` by default to protect user data on shipped devices
- Mark safe data `.public` to display it in logs
- Use `.private(mask: .hash)` to log a privacy-preserving hash that still allows correlation without revealing content

**Log Levels and Persistence**
- Five levels: `debug`, `info`, `notice` (default), `error`, `fault`
- Debug: fastest, never persisted, compiler skips message construction when not streaming
- Info: not persisted except near `log collect`
- Notice/Error/Fault: persisted (error/fault persist longer); error and fault highlighted in Console app
- Storage limit applies; older messages are purged automatically

**Collecting and Analyzing Logs**
- `log collect --device --start <time> --output <file.logarchive>` retrieves archived logs
- Console app: open `.logarchive`, filter by subsystem and keyword
- Xcode console streams logs in real time when device is connected
- Column-select with Option key in Console enables structured data export

**Formatting Options**
- `align: .left(columns:)` / `.right(columns:)` for fixed-width alignment
- `format: .fixed(precision:)` for floating-point rounding
- `format: .hex`, `.octal`, `.exponential` for numeric display
- Formatting is free at runtime — no additional cost per log call

## APIs & Frameworks

### os (Unified Logging — new Swift API in iOS 14)
- `Logger` **[NEW]** — Swift struct for structured logging; init with `subsystem:` and `category:`
- `Logger.log(_:)` **[NEW]** — logs at `.notice` (default) level
- `Logger.debug(_:)` **[NEW]** — logs at debug level; compiler-eliminated when not streaming
- `Logger.info(_:)` **[NEW]** — logs at info level
- `Logger.notice(_:)` **[NEW]** — explicit notice level
- `Logger.error(_:)` **[NEW]** — logs at error level (persisted, yellow in Console)
- `Logger.fault(_:)` **[NEW]** — logs at fault level (persisted longest, red in Console)
- `OSLogPrivacy` **[NEW]** — `.public`, `.private`, `.private(mask: .hash)` privacy options
- `OSLogStringAlignment` **[NEW]** — `.left(columns:)`, `.right(columns:)` alignment
- `OSLogFloatFormatting` **[NEW]** — `.fixed(precision:)`, `.hex`, `.exponential`, `.scientific`
- `os_log(_:_:)` — existing function; now accepts Swift string interpolations in iOS 14 **[UPDATED]**

### Supporting Types
- `CustomStringConvertible` — conform to include custom types in log messages

### Developer Tools
- `log collect` CLI — retrieves log archives from a connected device
- Console.app — opens `.logarchive` files; subsystem/keyword filtering; column selection

## Code Highlights

Basic logger setup and usage:
```swift
import os
let logger = Logger(subsystem: "com.example.Fruta", category: "giftcards")
logger.log("Started a task \(taskId, privacy: .public)")
logger.fault("Task \(currentTaskID, privacy: .public) is not running, cannot be stopped!")
```

Privacy with hash masking:
```swift
logger.log("Paid with bank account: \(accountNumber, privacy: .private(mask: .hash))")
```

Formatted statistics logging:
```swift
statisticsLogger.log("""
    \(taskID) \
    \(giftCardID, align: .left(columns: GiftCard.maxIDLength)) \
    \(serverID) \
    \(seconds, format: .fixed(precision: 2))
    """)
```

## Takeaways
- The new `Logger` API provides type-safe, compiler-optimized logging with zero string overhead at debug level — replace `print` statements with structured log calls.
- Privacy controls default to redacting non-numeric data, protecting users on shipped devices; use `.public` deliberately and `.private(mask: .hash)` for correlation without exposure.
- Log levels directly control both persistence and performance — use `debug` for verbose diagnostics and `fault` for programming errors/assumptions violated at runtime.
- `log collect` + Console app filtering by subsystem and runtime data (e.g., task UUIDs) enables post-hoc root cause analysis of bugs that are impossible to reproduce under a debugger.

---
_Source: WWDC20 Session 10168 page (abstract, chapter summaries, code samples, and resource links)._
