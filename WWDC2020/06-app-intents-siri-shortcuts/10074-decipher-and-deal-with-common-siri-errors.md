# Decipher and Deal with Common Siri Errors
**WWDC20 · Session 10074** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10074/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7

## Overview
This short, focused session covers practical techniques for debugging SiriKit intent extensions when things go wrong — specifically the dreaded "Sorry, there was a problem with the app" error. It explains how to automate Siri queries in Xcode, how to attach the debugger to multiple extension processes simultaneously, and how to identify the most common causes of intent failures.

The session is concise and tool-focused, providing a clear checklist of what to check when a Siri integration fails. The companion sample code is the "Soup Chef" app, which is referenced throughout the SiriKit and Shortcuts track.

## Key Topics

**Automating Siri Queries in Xcode**
Instead of invoking Siri manually and speaking your intent, you can pre-configure the Siri intent query in the Xcode scheme editor. This allows breakpoints and debugging to be triggered automatically without voice input, dramatically speeding up the test cycle.

**Attaching to the Correct Extension Process**
When launching an Intents extension from Xcode, you can choose between Siri and the Shortcuts app as the host process. Breakpoints in the Intents UI extension will not be hit if you are only attached to the Intents extension, because they run as separate processes. Use Xcode's Debug menu to attach to multiple processes simultaneously to debug both extensions at once.

**Common Causes of "Sorry, there was a problem with the app"**
1. **Completion handler timeout**: Intent handler protocol methods must call their completion handler within 10 seconds or Siri will time out and show the error.
2. **Completion handler called multiple times**: Calling a completion handler more than once throws an exception in the process. Each handler must be called exactly once.
3. **Extension process crash**: A crash in the Intents extension process produces this same generic error. Check Devices and Simulators → View Device Logs in Xcode to look for crash reports from your extension processes.

**Diagnosing Multi-Process Flows with `os_log` and Console.app**
SiriKit involves multiple processes (Siri/Shortcuts host, Intents extension, Intents UI extension, possibly the main app). Use `os_log` statements with a unique prefix (e.g., an emoji) to correlate log output from all processes. Filter by that keyword in Console.app to see a unified, accurate timeline of events across all processes.

## APIs & Frameworks

### SiriKit / Intents Framework
- `INIntentHandling` — protocol whose methods must call completion handlers exactly once, within 10 seconds
- Completion handler timeout — 10-second limit for all intent handler protocol methods
- Intents extension — separate process from the host app; attach debugger independently
- Intents UI extension — separate process from the Intents extension; requires separate debugger attachment

### Xcode Debugging Tools
- Scheme editor → Run → Arguments (Siri intent query) — automates Siri query without voice input
- Scheme editor → Run → Executable — choose Siri or Shortcuts app as the host process
- Xcode Debug menu → Attach to Process — attach to additional extension processes (Intents UI, main app)
- Devices and Simulators → View Device Logs — scan for crashes in extension processes

### Logging
- `os_log` / `Logger` (OSLog framework) — structured logging across extension processes
- Console.app — filter by subsystem, category, or keyword prefix to correlate multi-process logs

## Code Highlights

No code samples were shown in this session. The debugging techniques are tool- and workflow-based rather than API-based.

Best practice pattern for a completion handler (conceptual):
```swift
func handle(intent: OrderSoupIntent,
            completion: @escaping (OrderSoupIntentResponse) -> Void) {
    // Call completion exactly once, within 10 seconds
    guard let soup = intent.soup else {
        completion(OrderSoupIntentResponse(code: .failure, userActivity: nil))
        return
    }
    placeOrder(soup: soup) { result in
        // Still called exactly once on all code paths
        completion(OrderSoupIntentResponse(code: result ? .success : .failure,
                                           userActivity: nil))
    }
}
```

## Takeaways
- Configure the Siri intent query in the Xcode scheme editor to avoid manual voice invocation during debugging — this makes iterating on intent handlers dramatically faster.
- The "Sorry, there was a problem with the app" error almost always means: completion handler not called within 10 seconds, completion handler called more than once, or the extension process crashed.
- Always check Devices and Simulators → View Device Logs for crash reports from Intents extension processes; crashes surface as the generic Siri error with no other indication.
- Use `os_log` with a unique prefix and Console.app filtering to build a unified timeline across all SiriKit processes — critical for understanding handoffs between the Intents extension, UI extension, and host app.

---
_Source: WWDC20 Session 10074 page (abstract, chapter summaries, code samples, and resource links)._
