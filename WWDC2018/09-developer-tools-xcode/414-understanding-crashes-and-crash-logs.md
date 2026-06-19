# Understanding Crashes and Crash Logs
**WWDC18 · Session 414** · [Watch](https://developer.apple.com/videos/play/wwdc2018/414/)

_Platforms:_ iOS 12, macOS Mojave 10.14, watchOS 5, tvOS 12

## Overview
A three-part deep dive into crash analysis: why crashes happen, how to find and read crash logs, and how to track down hard-to-reproduce memory corruptions and threading races. The session uses a real "Chocolate Chip Cookies" recipe app as its running example, demonstrating every technique live in Xcode.

Part one (Chris) covers crash fundamentals and the Xcode Crashes Organizer — how to download TestFlight/App Store crash logs, open them alongside source code, and resolve issues. Part two (Greg) walks through the anatomy of a crash log text file, explains each section (exception type, thread stacks, registers, binary images), and demonstrates how to read memory-corruption signatures, including disassembling the compiler-generated ivar-destroyer function via lldb. Part three (Kuba) covers multithreading bugs: spotting them in crash logs, then using Thread Sanitizer to reproduce and fix a Swift dictionary data race.

## Key Topics

### Why Apps Crash
- **CPU limitation**: illegal instruction (SIGILL) — divide by zero, invalid opcode
- **OS policy**: watchdog timeouts (launch > 20s), memory pressure, thermal state, invalid code signature
- **Language runtime**: Swift array bounds, force-unwrap of nil, integer overflow — all generate `EXC_BAD_INSTRUCTION`
- **Developer assertions**: `precondition`, `assert`, `fatalError` in your own code

### Accessing Crash Logs
- **Crashes Organizer** (Xcode → Window → Organizer → Crashes): aggregates TestFlight + App Store crashes; groups by crash point; ranks by unique device count; supports all platforms including watchOS and app extensions; server-side symbolication requires uploading symbols with the build
- **Open in Project**: single click to open a crash log in the Debug Navigator alongside source code, with the crash line highlighted
- **Devices Window**: symbolicated using local dSYMs; shows all logs on a connected device
- **Test Results Bundle**: crash logs from test runs are automatically included and symbolicated
- **Console app**: for Simulator and macOS crashes
- **On-device**: Settings → Privacy → Analytics → Analytics Data (users can share logs directly)

### Symbolication Best Practices
1. Upload symbols with every App Store / TestFlight build (enabled by default; required for server-side symbolication)
2. Save app archives — Xcode uses Spotlight to find matching dSYMs for local symbolication
3. For bitcode builds: download recompiled dSYMs via Archives Organizer → Download Debug Symbols

### Reading a Crash Log (text file)
- **Header**: app name, version, OS version, date/time
- **Exception type + termination reason**: most important field — e.g., `EXC_BAD_INSTRUCTION` (SIGILL), `EXC_BAD_ACCESS` (SIGSEGV), `EXC_CRASH` (SIGKILL)
- **Application-specific information**: console log output, uncaught exception messages (hidden on iOS for privacy; available on Simulator and macOS)
- **Thread stacks**: all threads at time of crash; one marked as crash thread; look at ALL threads for multithreading clues
- **Register state + binary images**: used for symbolication and address analysis

### Exception Types
- `EXC_BAD_INSTRUCTION` / SIGILL — precondition / assertion failure (force unwrap, out-of-bounds, arithmetic overflow, `fatalError`)
- `EXC_BAD_ACCESS` / SIGSEGV — memory error: read from non-existent or write to read-only memory
- `EXC_CRASH` / SIGKILL — OS termination: launch timeout (`ate bad food`), memory pressure, thermal
- Unrecognized selector — commonly a use-after-free where a new object occupies the old address

### Memory Error Analysis
- **Use-after-free signature**: bad address in `EXC_BAD_ACCESS` is a valid malloc-region pointer rotated by 4 bits — malloc deliberately rotates free-list pointers to ensure freed objects crash on use
- **Reading the ivar destroyer**: compiler-generated `ivar_destroyer` function has no filename/line number, only a `+offset`; load crash log into lldb, disassemble the function, identify which property block contains the crashing offset
- **Other memory crash patterns**: crash inside `objc_release`, `objc_msgSend`, `malloc`, `free`, `_objc_dealloc` machinery; `malloc` calling `abort` signals heap corruption or double-free
- **Strategy**: narrow down to the affected object/property from crash log, then use Address Sanitizer or Zombies instrument to reproduce

### Multithreading Bugs
- **Crash log clues**: same class/method appears on multiple threads; crashes at slightly different addresses across logs (same crash appears as multiple crash groups); crash thread may not be the bug source
- **Thread Sanitizer**: detects Swift access races and data races; reproduces issues extremely reliably (no need for repeated manual triggering); works on macOS and in Simulator
- Enable: Xcode → Product → Scheme → Edit Scheme → Diagnostics → Thread Sanitizer + Pause on Issues
- **Fix**: protect shared mutable state; use `DispatchQueue` (serial by default) with `queue.sync { }` around all accesses — both getter and setter must be synchronized, not just the setter

### Watchdog / Launch Timeout
- Watchdog disabled in Simulator and with debugger attached — test without debugger using TestFlight or iOS App Launcher
- Test on real devices, especially the oldest supported hardware
- `EXC_CRASH` + `SIGKILL` + termination reason `0x8badf00d` + "exhausted real clock time allowance of N seconds"

## APIs & Frameworks

**Xcode Tools**
- **Crashes Organizer** — `Window → Organizer → Crashes`; download and browse crash logs from TestFlight and App Store
- **Open in Project** button — opens crash log in Debug Navigator alongside source
- **Devices Window** — local device crash log viewer
- **lldb crash log commands** — `command script import lldb.macosx.crashlog`; `crashlog <path>` to import crash log into debugging session; `disassemble -n <function>` to read assembly

**Diagnostics (Scheme Editor → Diagnostics)**
- **Thread Sanitizer** — detects data races and Swift access races; use with Pause on Issues
- **Address Sanitizer** — detects buffer overflows and use-after-free
- **Zombie Objects** — detects use-after-free of Objective-C objects

**Swift Language**
- `CaseIterable` **[NEW in Swift 4.2]** — auto-synthesizes `allCases` array; use `RecipeSection.allCases.count` instead of hardcoded counts in `numberOfSections`
- `DispatchQueue(label:)` — create a serial queue to protect shared mutable state; use `queue.sync { }` for thread-safe access

**Foundation/Dispatch**
- `DispatchQueue` — `sync(_:)` for synchronous execution; serial queue ensures mutual exclusion
- `NSArray` / Swift `Array` — both assert on out-of-bounds access and generate `EXC_BAD_INSTRUCTION`

**References**
- [Understanding and Analyzing Application Crash Reports (TN2151)](https://developer.apple.com/library/archive/technotes/tn2151/_index.html)
- [iOS Debugging Magic (TN2239)](https://developer.apple.com/library/archive/technotes/tn2239/_index.html)
- [Mac OS X Debugging Magic (TN2124)](https://developer.apple.com/library/archive/technotes/tn2124/_index.html)

## Code Highlights

Fix for enum-based crash using `CaseIterable` (Swift 4.2):
```swift
enum RecipeSection: Int, CaseIterable {
    case ingredients = 0
    case steps = 1
}

// Table view data source — was returning ingredients.count (wrong):
func numberOfSections(in tableView: UITableView) -> Int {
    return RecipeSection.allCases.count  // returns 2, always correct
}
```

Thread-safe image cache using a serial DispatchQueue:
```swift
class ImageCache {
    private var storage = [String: UIImage]()
    private let queue = DispatchQueue(label: "com.example.ImageCache")

    subscript(key: String) -> UIImage? {
        get { return queue.sync { storage[key] } }
        set { queue.sync { storage[key] = newValue } }
    }
}
```

Loading a crash log into lldb for disassembly:
```
(lldb) command script import lldb.macosx.crashlog
(lldb) crashlog /path/to/crash.ips
(lldb) disassemble -n "LoginViewController.ivar_destroyer"
```

## Takeaways
- Always test on real devices without the debugger attached before shipping — watchdog timeouts are invisible in the Simulator and debugger.
- The crash thread is not always the bug source: read all thread stacks, especially for memory corruption and threading issues.
- A bad address matching the malloc range but rotated 4 bits is the malloc free-list signature — a strong indicator of use-after-free.
- Thread Sanitizer reproduces data races reliably on the first try; enable it routinely during development on any code touching shared mutable state.

---
_Source: WWDC18 Session 414 page (abstract, full transcript, and resource links)._
