# Meet Shortcuts for macOS
**WWDC21 · Session 10232** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10232/)

_Platforms:_ macOS Monterey 12, iOS 15

## Overview
Shortcuts arrives on macOS Monterey, bringing the same visual automation builder from iOS to the Mac with a native AppKit/SwiftUI redesign. Shortcuts on Mac syncs shortcuts from iPhone via iCloud, hosts a Gallery of Mac-specific shortcuts, and integrates deeply with the existing Mac automation ecosystem — including AppleScript, Shell Scripts, and Automator. Third-party apps expose functionality as Shortcuts actions using the same Intents (SiriKit) framework already used on iOS.

This session covers how to build Shortcuts actions for macOS apps (AppKit or Mac Catalyst), how Shortcuts interacts with files and cross-platform deployments, Automator migration, and how to run shortcuts programmatically from other apps or command-line tools.

## Key Topics

**Shortcuts on macOS**
Shortcuts app on macOS Monterey is written almost entirely in SwiftUI. It includes a full editor, a Gallery with Mac-specific shortcuts, and a menu bar item for quick invocation. Shortcuts can also be triggered via keyboard shortcut, Spotlight, and Finder Quick Actions.

**Automator Migration**
A built-in migrator converts most existing Automator workflows to Shortcuts. Users drag a workflow file into the Shortcuts app (or right-click) and each Automator action maps to one or more Shortcuts actions. Popular Automator actions (Shell Scripts, AppleScripts, file management) are all present in Shortcuts to support this migration path.

**AppleScript and Shell Script Support**
Shortcuts includes built-in actions to write and run AppleScripts and Shell Scripts directly in the editor.

**Sharing Improvements**
- iCloud links — notarized by Apple; distributable from a website or app.
- Shortcuts files (`.shortcut`) — new file format for sharing outside iCloud; also notarized.
- Private sharing — signed with the sender's identity, no iCloud upload required.
- `shortcuts` command-line tool — signs/re-signs Shortcuts files.

**Building Actions with the Intents Framework**
Actions for Shortcuts use the same Intents framework (SiriKit) as iOS. The workflow:
1. Create an Intent Definition file (`.intentdefinition`) in Xcode and add it to the app target.
2. Define custom types (e.g., `Task`) and intent definitions (e.g., `CreateTaskIntent`) with parameters.
3. Xcode generates Swift types and handling protocols automatically.
4. Implement an intent handler conforming to the generated `*IntentHandling` protocol.
5. Dispatch the intent handler from `NSApplicationDelegate.application(_:handlerFor:)` (in-app) or an `INExtension` subclass (extension).

**Intent Handler Method Types**
- `resolve*(for:with:)` — validate each parameter; return `.success`, `.needsValue`, or `.unsupported(forReason:)`.
- `provideOptions*(for:with:)` — enumerate dynamic option values when a parameter uses Dynamic Options.
- `confirm(intent:completion:)` — final pre-flight check (e.g., network reachability).
- `handle(intent:completion:)` — perform the action and return a response with output values.

**Mac Catalyst Considerations**
Catalyst apps that disabled Intents integration on macOS must re-enable it for macOS Monterey — `NSApplicationDelegate` now exposes `application(_:handlerFor:)` like its UIKit counterpart.

**File Parameters**
iOS 15 and macOS Monterey add file parameters for intents, letting users select specific files and pass them into actions. Document-based apps should expose file-centric actions (open, process, export).

**Running Shortcuts Programmatically**
Apps can list and run shortcuts via the "Shortcuts Events" AppleScript target or via `ScriptingBridge` in Swift/Objective-C. The `shortcuts` command-line tool supports `list` and `run` for shell scripts and CI. Sandboxed apps require the `com.apple.security.scripting-targets` entitlement with the `com.apple.shortcuts.run` target.

## APIs & Frameworks

### Intents Framework (macOS Monterey additions) **[NEW]**
- `NSApplicationDelegate.application(_:handlerFor:)` — in-app intent dispatch on macOS **[NEW]**
- Intent Definition file (`.intentdefinition`) — Xcode source file; compile-time code generation
- `INExtension.handler(for:)` — override in an Intents Extension for out-of-process handling
- `INStringResolutionResult.needsValue()` — prompt user for a missing string parameter
- `INStringResolutionResult.success(with:)` — accept a valid string value
- Custom resolution result (e.g., `CreateTaskDueDateResolutionResult`) — generated per intent/parameter
  - `.unsupported(forReason:)` — reject with a custom error code defined in the intent definition

### Shortcuts App Integration **[NEW on macOS]**
- Shortcuts Gallery — discoverable Mac-specific starter shortcuts **[NEW]**
- Menu bar shortcut runner — run shortcuts without opening the app **[NEW]**
- Spotlight / Keyboard Shortcut invocation of Shortcuts actions **[NEW]**

### Shortcuts Sharing **[NEW]**
- `.shortcut` file format — notarized shortcut file for distribution outside iCloud **[NEW]**
- `shortcuts` CLI tool — `shortcuts list`, `shortcuts run "<name>"`, file signing **[NEW]**

### ScriptingBridge / AppleScript **[NEW Shortcuts integration]**
- `SBApplication(bundleIdentifier: "com.apple.shortcuts.events")` — access Shortcuts Events process **[NEW]**
- `ShortcutsEvents.shortcuts` — `SBElementArray` of user's shortcuts **[NEW]**
- `Shortcut.run(withInput:)` — execute a shortcut and return output **[NEW]**
- Entitlement: `com.apple.security.scripting-targets` → `com.apple.shortcuts.run` (sandbox apps)

## Code Highlights

NSApplicationDelegate dispatch for in-app intent handling:
```swift
import Intents

class AppDelegate: NSObject, NSApplicationDelegate {
    func application(_ application: NSApplication,
                     handlerFor intent: INIntent) -> Any? {
        if intent is CreateTaskIntent {
            return IntentHandler()
        }
        return nil
    }
}
```

Resolve and validate a parameter:
```swift
func resolveDueDate(for intent: CreateTaskIntent,
                    with completion: @escaping (CreateTaskDueDateResolutionResult) -> Void) {
    guard let dateComponents = intent.dueDate,
          let dueDate = Calendar.current.date(from: dateComponents) else {
        return completion(.needsValue())
    }
    if dueDate < Date() {
        return completion(.unsupported(forReason: .invalidDate))
    }
    return completion(.success(with: dateComponents))
}
```

Handle the intent and return output:
```swift
func handle(intent: CreateTaskIntent,
            completion: @escaping (CreateTaskIntentResponse) -> Void) {
    let task = createTask(name: intent.title!, due: intent.dueDate!)
    let response = CreateTaskIntentResponse(code: .success, userActivity: nil)
    response.task = task
    completion(response)
}
```

Run a shortcut from Swift via ScriptingBridge:
```swift
import ScriptingBridge

guard let app: ShortcutsEvents = SBApplication(bundleIdentifier: "com.apple.shortcuts.events"),
      let shortcut = app.shortcuts?.object(withName: "Make GIF") as? Shortcut else { return }
_ = shortcut.run?(withInput: nil)
```

Run a shortcut from the command line:
```bash
shortcuts run "Make GIF"
```

## Takeaways
- Shortcuts on macOS Monterey uses the same SiriKit Intents framework as iOS — adding `NSApplicationDelegate.application(_:handlerFor:)` is the only Mac-specific change needed for most apps.
- Automator users can migrate existing workflows by dragging `.workflow` files into the Shortcuts app; supporting this migration by exposing equivalent Shortcuts actions benefits existing Automator power users.
- Cross-platform apps should compile the same `.intentdefinition` file to both iOS and macOS targets with matching intent names so shortcuts built on one platform run on the other.
- The new `.shortcut` file format and `shortcuts` CLI tool enable notarized, serverless distribution of shortcuts from any website or app.

---
_Source: WWDC21 Session 10232 page (abstract, chapter summaries, code samples, and resource links)._
