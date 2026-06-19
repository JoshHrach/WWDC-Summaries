# Widgets Code-along, Part 3: Advancing Timelines
**WWDC20 · Session 10036** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10036/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
The third and final part of the Widgets Code-along covers advanced WidgetKit topics: making network requests (both in-process URL sessions and background URL sessions) from the timeline provider, using SwiftUI's `Link` API to create independently tappable regions within medium and large widgets, combining multiple widget types into a single extension via `@main WidgetBundle`, and providing a dynamic (server-driven) set of configuration options using an Intents Extension instead of a hard-coded enum.

## Key Topics

### URL Sessions in Widget Extensions
The `TimelineProvider` API uses completion handlers (not return values) specifically to accommodate asynchronous work like network fetches:
- **In-process URL sessions** — use `URLSession` exactly as in an app; the data task calls the completion handler which then calls the timeline provider's completion
- **Background URL sessions** — widgets handle their own background sessions (unlike other extensions that hand off to the host app)

The `.onBackgroundURLSessionEvents(matching:_:)` modifier on `WidgetConfiguration` replaces the `UIApplicationDelegate.application(_:handleEventsForBackgroundURLSession:completionHandler:)` method:
- Provides the `sessionIdentifier` and a `completionHandler` closure
- Manage them the same as in an app (store the completion handler, call it after processing)

### SwiftUI Link for Tappable Widget Regions
`systemSmall` widgets use `.widgetURL(_:)` for a single whole-widget deep link. For `systemMedium` and `systemLarge`, wrap any subview in SwiftUI's `Link(destination: url)` to make that region independently tappable:
```swift
Link(destination: character.url) {
    CharacterRow(character: character)
}
```
Each row in a leaderboard can link to its own character's detail screen.

### Widget Bundles
A single widget extension can vend multiple widget types. Remove `@main` from individual `Widget` structs and create a new struct conforming to `WidgetBundle` tagged with `@main`:
```swift
@main
struct EmojiRangerWidgetBundle: WidgetBundle {
    @WidgetBundleBuilder
    var body: some Widget {
        EmojiRangerWidget()
        LeaderboardWidget()
    }
}
```
The widget gallery shows all widget types from the bundle; users can add any combination.

### Dynamic Intent Configuration (Intents Extension)
Hard-coded enum options in the Intent Definition become a custom type (`identifier` + `displayString`) whose values are supplied at runtime by an Intents Extension:
1. Add an Intents Extension target (File > New > Target > Intents Extension; starting point: None)
2. In the Intent Handler, implement the resolution method (e.g., `provideHeroOptionsCollection`) as an async call that fetches options from the network or a data store
3. In the widget, switch from the enum-based property to `configuration.hero?.identifier` (a string) to look up the selected character
4. The Edit Widget sheet automatically fetches dynamic options from the Intents Extension at configuration time

## APIs & Frameworks

### WidgetKit
- `WidgetConfiguration.onBackgroundURLSessionEvents(matching:_:)` **[NEW]** — modifier replacing the app delegate background URL session method; provides `sessionIdentifier: String` and `completionHandler: () -> Void`
- `@main struct ...: WidgetBundle` **[NEW]** — declares a bundle of multiple widget types from one extension
- `@WidgetBundleBuilder` **[NEW]** — result builder for `WidgetBundle.body`

### SwiftUI
- `Link(destination: URL, label: () -> View)` **[NEW]** — creates a tappable region within `systemMedium`/`systemLarge` widgets that deep-links into the app; each `Link` within the widget is independently tappable

### SiriKit / Intents Extension
- Intents Extension target — `INExtension` subclass implementing resolution handlers
- Dynamic `INObjectCollection<CharacterType>` returned from resolution handler — populates Edit Widget picker with server-driven options

## Code Highlights

Background URL session handling in a widget:
```swift
struct LeaderboardWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: "Leaderboard", provider: LeaderboardProvider()) { entry in
            LeaderboardWidgetEntryView(entry: entry)
        }
        .onBackgroundURLSessionEvents(matching: "com.example.leaderboard") { identifier, completionHandler in
            // Store completionHandler; call it after URLSession delegate finishes
            BackgroundSessionManager.shared.handle(identifier: identifier,
                                                   completionHandler: completionHandler)
        }
    }
}
```

SwiftUI Link inside a medium/large widget row:
```swift
struct AllCharactersView: View {
    var characters: [CharacterDetail]
    var body: some View {
        VStack {
            ForEach(characters) { character in
                Link(destination: character.url) {
                    CharacterRow(character: character)
                }
            }
        }
    }
}
```

Widget bundle with two widget types:
```swift
@main
struct EmojiRangerWidgetBundle: WidgetBundle {
    @WidgetBundleBuilder
    var body: some Widget {
        EmojiRangerWidget()
        LeaderboardWidget()
    }
}
```

Dynamic intent resolution in the Intents Extension:
```swift
class IntentHandler: INExtension, CharacterSelectionIntentHandling {
    func provideHeroOptionsCollection(
        for intent: CharacterSelectionIntent,
        with completion: @escaping (INObjectCollection<CharacterType>?, Error?) -> Void
    ) {
        // Fetch from network or data store; return dynamic options
        let characters = CharacterDetail.remoteCharacters.map {
            CharacterType(identifier: $0.name, display: $0.name)
        }
        completion(INObjectCollection(items: characters), nil)
    }
}
```

## Takeaways

- Widget extensions fully own their background URL sessions via `onBackgroundURLSessionEvents` — there is no host-app involvement, so background networking works entirely within the widget process.
- `Link` within a medium or large widget turns any SwiftUI subview into an independent deep link; use this for leaderboards, lists, or multi-item widgets where each item navigates to different content.
- `@main WidgetBundle` is the correct way to ship multiple widget types from one extension; the gallery automatically lists all widgets in the bundle.
- Dynamic intent options from an Intents Extension let the widget's configuration UI show server-populated choices (e.g., recently played characters, followed teams) rather than a static compile-time list.

---
_Source: WWDC20 Session 10036 page (abstract, transcript, and resource links)._
