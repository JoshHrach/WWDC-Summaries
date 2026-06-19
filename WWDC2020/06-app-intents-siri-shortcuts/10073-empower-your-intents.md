# Empower Your Intents
**WWDC20 · Session 10073** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10073/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
This session covers practical techniques for optimizing SiriKit intent implementations in iOS 14. The two headline additions are in-app intent handling (a new alternative to Intents extensions) and rich disambiguation (images and subtitles in disambiguation lists). Additional coverage includes dynamic search for intent parameters, the new `INObjectCollection` for grouped dynamic options, per-parameter configurable/resolvable flags, intent deprecation, and several Xcode workflow tips for managing custom intent code generation.

## Key Topics

### In-App Intent Handling **[NEW in iOS 14]**
Previously, resolving and confirming always ran in an Intents extension; the handle phase could trigger the app via `continueInApp`. iOS 14 allows the entire resolve/confirm/handle lifecycle to run inside the app process, which is useful when:
- The intent directly controls live on-screen UI (e.g., "next step" in a cooking directions view).
- The task requires media or workout session control that was already split across extension + app.
- The intent requires high-memory processing (photo/video) that would exceed extension memory limits.
- The codebase cannot easily factor shared code into a framework or extension.

**Trade-off**: the app's launch time counts against the 10-second intent timeout. Prefer an Intents extension when one can handle the task — extensions link fewer frameworks and launch faster.

**Implementation requirements**:
1. The app must adopt the `UIScene` lifecycle with multi-window support.
2. List supported intents in the target's "Supported Intents" section in Xcode.
3. Implement `application(_:handlerFor:)` in `UIApplicationDelegate` — return the handler object for each intent type.
4. If the intent affects on-screen UI and the app is backgrounded, respond with `.continueInApp` in the handle phase and handle the resulting `NSUserActivity` in `UISceneDelegate.scene(_:willConnectTo:options:)` and `scene(_:continue:)`.

### Intents Extension Launch Time Optimization
The 10-second timeout starts when the user's request initiates a connection to the extension — before any business logic runs. Key optimization: link only the frameworks the extension actually needs. Every framework linked must load before any handler code runs.

### Rich Disambiguation **[NEW in iOS 14]**
Disambiguation lists (shown when resolving intent parameters) can now include:
- **Subtitle** — a `String` shown below the display name.
- **Image** — an `INImage` shown alongside the item.

These apply to both runtime disambiguation responses and dynamic options in the Shortcuts app. Siri's voice-only disambiguation now supports **pagination**: developers specify the maximum number of items spoken at once and can provide a subsequent introduction string (configured per-parameter in the intent definition file's Siri Dialog section).

### Dynamic Search for Dynamic Options **[NEW in iOS 14]**
The dynamic options API (introduced in iOS 13) is extended with a `searchTerm` parameter. A new checkbox in the intent editor — "Provide search results as you type" — code-generates a new provider method that receives the current search term and is called on each keystroke. If the search term is empty, return a default list. Use only for large catalogs; the Shortcuts app provides built-in filtering for small static lists.

### `INObjectCollection` **[NEW]**
The completion handler for the dynamic options provider method now accepts `INObjectCollection<T>` instead of a flat array. `INObjectCollection` supports:
- Grouped sections with titles.
- Optional indexed collation for alphabetical browsing.

### Configurable vs. Resolvable Parameters **[NEW in iOS 14]**
Parameters can now be independently marked as **configurable** (user-facing in Shortcuts editor) and **resolvable** (requires runtime resolution by the handler). Parameters marked as non-resolvable are never resolved by Siri or the Shortcuts app, eliminating the need to provide Siri dialog for them. Previously, all user-facing parameters required resolution.

### Intent Deprecation **[NEW in Xcode 12]**
Custom intents can be marked as deprecated in the intent editor (check "Deprecated" in the Intent inspector). Effect: existing shortcuts using the intent show a "may not be available in future versions" warning in the Shortcuts app; the intent is hidden from the Shortcuts action picker.

### Xcode Workflow Tips
- **Custom class names**: specify in the "Custom Class" inspector field per intent/type/enum, or set a project-wide class prefix in the Project Document inspector. The type name in the editor is not the class name.
- **Intents UI extension membership**: create a separate intent definition file for intents that do not need a custom UI and exclude it from the Intents UI extension target.
- **Code generation language**: override per-target in Build Settings to force Objective-C or Swift generation, rather than relying on Xcode's automatic detection.

## APIs & Frameworks

### SiriKit / Intents (iOS 14)
- `UIApplicationDelegate.application(_:handlerFor:)` **[NEW]** — returns intent handler from the app process
- `INObjectCollection<ObjectType>` **[NEW]** — grouped dynamic options with sections and optional indexed collation
- `INImage` — attach to custom type `displayImage` property for rich disambiguation
- Dynamic options provider method with `searchTerm: String` **[NEW]** — generated when "Provide search results as you type" is enabled
- Disambiguation pagination — configured in intent definition file's Siri Dialog section (max items, subsequent introduction)
- Per-parameter "Configurable" and "Resolvable" flags **[NEW]** — independent control over parameter behavior
- `continueInApp` handle response code — triggers `NSUserActivity` in `UISceneDelegate`
- Intent deprecation — "Deprecated" checkbox in Xcode intent editor **[NEW in Xcode 12]**

### UIKit (prerequisite for in-app intent handling)
- `UIScene` lifecycle with multi-window support (see WWDC19 "Introducing Multiple Windows on iPad")
- `UISceneDelegate.scene(_:willConnectTo:options:)` — handle launch from Siri
- `UISceneDelegate.scene(_:continue:)` — handle `continueInApp` user activity

## Code Highlights

App delegate returning in-app intent handler:
```swift
// UIApplicationDelegate
func application(_ application: UIApplication,
                 handlerFor intent: INIntent) -> Any? {
    if intent is ShowDirectionsIntent {
        return IntentHandler.current
    }
    return nil
}
```

Checking foreground state and responding with `continueInApp`:
```swift
func handle(intent: ShowDirectionsIntent,
            completion: @escaping (ShowDirectionsIntentResponse) -> Void) {
    guard UIApplication.shared.applicationState != .background else {
        completion(ShowDirectionsIntentResponse(code: .continueInApp, userActivity: nil))
        return
    }
    nextStepProvider?.nextStep()
    completion(ShowDirectionsIntentResponse(code: .success, userActivity: nil))
}
```

Providing grouped dynamic options with `INObjectCollection`:
```swift
func provideRecipeOptionsCollection(
    for intent: ShowDirectionsIntent,
    searchTerm: String?,
    with completion: @escaping (INObjectCollection<Recipe>?, Error?) -> Void
) {
    let results = searchTerm.map { catalog.search($0) } ?? catalog.featured
    let collection = INObjectCollection(sections: [
        INObjectSection(title: "Featured", items: results)
    ])
    completion(collection, nil)
}
```

## Takeaways
- In-app intent handling is ideal when an intent needs to directly manipulate on-screen UI, requires heavy processing, or when factoring code into an extension is impractical — but always weigh the app's launch time cost against the 10-second timeout.
- Rich disambiguation (subtitles + images) significantly improves the voice and visual disambiguation experience with minimal API surface — add `displayImage` and `displaySubtitle` to custom intent types.
- Mark parameters as non-resolvable when Siri or the Shortcuts app should not prompt for them at runtime, reducing conversational overhead.
- Dynamic search with `searchTerm` enables type-ahead catalog search in the Shortcuts editor — use it only for large catalogs, not small lists.

---
_Source: WWDC20 Session 10073 page (abstract, transcript, and resource links)._
