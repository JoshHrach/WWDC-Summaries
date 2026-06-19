# Make Features Discoverable with TipKit
**WWDC23 · Session 10229** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10229/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10

## Overview
TipKit is a new framework that makes it simple for apps to surface contextual, educational tips to users. Rather than building custom coaching flows, developers define tips as Swift types conforming to the `Tip` protocol, configure eligibility via predicate-based rules, and let TipKit control display frequency, persistence, and dismissal — all synchronized via iCloud so a tip seen on one device won't reappear on another.

Tips are intended for instructional, feature-discovery content: teaching a user about a hidden feature, showing a faster workflow, or highlighting a brand-new capability. They are not appropriate for promotional messages, error states, or generic announcements. TipKit is available on iPhone, iPad, Mac, Apple Watch, and Apple TV.

## Key Topics

### Defining a Tip
- Conform a struct to the `Tip` protocol.
- Required: `title` (a `Text` view) and `message` (a `Text` view).
- Optional: `asset` (an `Image` for an icon), `actions` (buttons: link to settings or onboarding), and `rules`.
- Tips persist between app launches via `TipsCenter`.

### Tip Views (Presentation Styles)
- **Popover view** (`.popoverMiniTip(tip:)`) — overlays the app's UI and points directly at a button or element. Exclusive presentation style on tvOS.
- **Inline view** (`TipView`) — temporarily adjusts the app's UI layout to embed the tip, ensuring no UI is blocked.
- Placement near the relevant button improves discoverability and reduces cognitive load.

### Eligibility Rules
Two rule types compose to target the ideal audience:

**Parameter-based rules:**
- Use `@Parameter` property wrapper on a static stored property in the tip struct.
- Predicate expression written with `#Rule(Self.$parameter)` macros.
- Persisted between launches; update the parameter value anywhere in the app.

**Event-based rules:**
- Define a static `Event` property with an ID string.
- Write rules using `#Rule(Self.event)` with count queries.
- Donate events from anywhere in the app: `MyTip.someEvent.donate()`.
- Filter donations by date: `$0.donations.filter { $0.date > cutoff }.count >= N`.
- Custom donation types: conform to `DonationValue`; donate associated data (e.g., a `backyardID: Int`); query with `.largestSubset(by:)` to count distinct values.
- Keep donation payloads small; larger data slows queries.

### Display Frequency
Set at the `TipsCenter` level in `configure {}`:
- `.daily` — one tip per 24 hours.
- `.hourly` — one tip per 60 minutes.
- `DisplayFrequency(customTimeInterval)` — any `TimeInterval`.
- `.immediate` — no frequency throttle; show tips as soon as eligible.

Per-tip override: add `.ignoresDisplayFrequency(true)` to the tip's `options` array to bypass the global frequency for that specific tip.

### Dismissal / Invalidation
- `tip.invalidate(reason: .userPerformedAction)` — call when the user performs the described action; tip is permanently dismissed.
- `.maxDisplayCount(N)` option — tip auto-dismisses after being shown N times without user action.
- `tip.invalidate(reason: .tipClosed)` — user explicitly closes the tip.
- Custom criteria: call `invalidate()` based on any app-defined condition.

### iCloud Sync
- Tip status syncs across devices via iCloud.
- A tip seen and dismissed on iPhone will not reappear on the user's iPad (when the feature is the same on both devices).

### Testing APIs
All testing APIs call on `TipsCenter`:
- `TipsCenter.showAllTips()` — show all tips regardless of eligibility.
- `TipsCenter.showTips([tip1, tip2])` — show specific tips.
- `TipsCenter.hideTips([tip1])` — hide specific tips.
- `TipsCenter.hideAllTips()` — hide all tips.
- `TipsCenter.resetDatastore()` — purge all TipKit data; resets to pristine state at each build.

Equivalent Xcode scheme launch arguments:
- `com.apple.TipKit.ShowAllTips 1`
- `com.apple.TipKit.ShowTips tipID,otherTipID`
- `com.apple.TipKit.HideAllTips 1`
- `com.apple.TipKit.HideTips tipID,otherTipID`
- `com.apple.TipKit.ResetDatastore 1`

## APIs & Frameworks
- `TipKit` framework **[NEW]** — in-app feature discovery and tip management
- `Tip` protocol **[NEW]** — defines a tip; `title`, `message`, `asset`, `actions`, `rules`, `options`
- `TipsCenter` **[NEW]** — shared singleton; configures and manages all tips
- `TipsCenter.shared.configure(_:)` **[NEW]** — configures TipKit with display frequency and options
- `DisplayFrequency` **[NEW]** — configures global tip display cadence (`.daily`, `.hourly`, `.immediate`, custom `TimeInterval`)
- `@Parameter` **[NEW]** — property wrapper for parameter-based rule values; persistent across launches
- `#Rule(_:_:)` **[NEW]** — macro for defining eligibility predicates
- `Event` **[NEW]** — event type for event-based rules; identified by string ID
- `Event.donate()` **[NEW]** — records an event occurrence
- `Event.donate(with:)` **[NEW]** — records an event occurrence with associated `DonationValue` data
- `DonationValue` **[NEW]** — protocol for custom event donation payloads
- `Tip.Action` **[NEW]** — action button in a tip (ID + title)
- `Tip.Option` **[NEW]** — tip option enum
- `.ignoresDisplayFrequency(true)` **[NEW]** — per-tip option to bypass global display frequency
- `.maxDisplayCount(N)` **[NEW]** — per-tip option to auto-dismiss after N displays
- `Tip.invalidate(reason:)` **[NEW]** — dismisses a tip with a reason
- `TipInvalidationReason.userPerformedAction` **[NEW]** — invalidation reason: user performed the described action
- `TipInvalidationReason.tipClosed` **[NEW]** — invalidation reason: user explicitly closed the tip
- `.popoverMiniTip(tip:)` **[NEW]** — view modifier to attach a popover tip to a view
- `TipView` **[NEW]** — inline tip view that adjusts surrounding layout
- `TipsCenter.showAllTips()` **[NEW]** — testing: show all tips
- `TipsCenter.showTips(_:)` **[NEW]** — testing: show specific tips
- `TipsCenter.hideTips(_:)` **[NEW]** — testing: hide specific tips
- `TipsCenter.hideAllTips()` **[NEW]** — testing: hide all tips
- `TipsCenter.resetDatastore()` **[NEW]** — testing: purge all TipKit state

## Code Highlights

Define a tip struct:
```swift
struct FavoriteBackyardTip: Tip {
    var title: Text { Text("Save as a Favorite").foregroundColor(.indigo) }
    var message: Text { Text("Your favorite backyards always appear at the top of the list.") }
    var asset: Image { Image(systemName: "star") }
    var actions: [Action] {
        [Tip.Action(id: "learn-more", title: "Learn More")]
    }
}
```

Configure TipsCenter in the app entry point:
```swift
init() {
    TipsCenter.shared.configure {
        DisplayFrequency(.daily)
    }
}
```

Attach a popover tip to a toolbar button:
```swift
.toolbar {
    ToolbarItem {
        Button { backyard.isFavorite.toggle() } label: {
            Label("Favorite", systemImage: "star")
        }
        .popoverMiniTip(tip: favoriteBackyardTip)
    }
}
```

Parameter + event-based eligibility rules with date filtering:
```swift
struct FavoriteBackyardTip: Tip {
    @Parameter static var isLoggedIn: Bool = false
    static let enteredBackyardDetailView = Event<DetailViewDonation>(
        id: "entered-backyard-detail-view"
    )

    var rules: Predicate<RuleInput...> {
        #Rule(Self.$isLoggedIn) { $0 == true }
        #Rule(Self.enteredBackyardDetailView) {
            $0.donations.filter {
                $0.date > Date.now.addingTimeInterval(-5 * 60 * 60 * 24)
            }.count >= 3
        }
    }
    var options: [Option] { [.maxDisplayCount(5)] }
}
```

Donate an event with custom payload:
```swift
.onAppear {
    FavoriteBackyardTip.enteredBackyardDetailView.donate(
        with: .init(backyardID: backyard.id)
    )
}
```

Invalidate a tip when the user performs the action:
```swift
Button {
    backyard.isFavorite.toggle()
    favoriteBackyardTip.invalidate(reason: .userPerformedAction)
} label: {
    Label("Favorite", systemImage: "star")
}
```

## Takeaways
- TipKit reduces in-app education to conforming to the `Tip` protocol and calling `TipsCenter.shared.configure()` in the app initializer — the framework handles persistence, frequency throttling, and iCloud sync automatically.
- Combine parameter-based rules (static state, e.g., "is user logged in?") with event-based rules (behavioral counts with date windows) to precisely target the users most likely to benefit from a tip.
- `TipsCenter.resetDatastore()` (or the `com.apple.TipKit.ResetDatastore 1` launch argument) is essential during development to test eligibility from a clean state on every build.
- Tips should be short, actionable, and instructional — they are not suitable for promotional messages, error states, or passive announcements.

---
_Source: WWDC23 Session 10229 page (abstract, transcript, chapter summaries, and code samples)._
