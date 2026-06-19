# What's new in Xcode 15
**WWDC23 · Session 10165** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10165/)

_Platforms:_ macOS Sonoma 14 (Xcode host)

## Overview
Xcode 15 delivers substantial productivity improvements across the entire development workflow: smarter code completion with file-name inference and context awareness, asset catalog Swift symbol generation, String Catalogs for centralized localization, a real-time documentation preview, Swift macro integration with expand-in-place, a new `#Preview` macro for SwiftUI/UIKit/AppKit, a Bookmarks navigator, a redesigned source control commit editor, a rewritten test navigator with 45% performance gain and automation explorer playback, full OSLog integration in the debug console, and streamlined distribution with privacy manifests, framework signature verification, and TestFlight internal testing.

## Key Topics

**Editing**
- Code completion improvements: file-name inference (struct name = file name), full permutation display for default arguments, top suggestion considers context (most-used modifier, sibling properties like latitude/longitude)
- Asset catalogs: color and image assets now generate Swift symbols → compile-time type safety, autocomplete, and build-time errors on rename
- String Catalogs **[NEW]**: `.xcstrings` format merges `.strings` + `.stringsdict`; Convert via Edit > Convert to String Catalog; strings pulled from source at build time; per-language progress in sidebar; auto-annotates new/removed strings
- Swift-DocC: new real-time Documentation Preview assistant (Editor > Assistant > Documentation Preview); updates as you type; shows screenshots from documentation catalog
- Swift Macros in Xcode: "Expand Macro" via Quick Actions (Cmd-Shift-A); breakpoints inside expanded macro code; packages scaffolded with `Swift Macro Package` template

**Previews (NEW `#Preview` macro)**
- New `#Preview { ... }` macro syntax — clean, no struct needed
- Supports named previews: `#Preview("Placeholder View") { ... }`
- UIKit and AppKit view controller previews now supported
- Widget previews: `#Preview(as: .systemSmall) { ... } timeline: { ... }` with timeline navigation and animation playback in canvas

**Navigation**
- Bookmarks navigator **[NEW]**: bookmark code lines, search queries; add description; group bookmarks; mark complete; delete; "New Group From Selection"; accessible via navigator bar
- Quick Actions (Cmd-Shift-A) — new command palette for all Xcode menu items

**Source Control**
- New Changes navigator with improved file status icons
- New commit editor: single scrolling view of all modifications with drag-handle context expansion; inline editing during review; per-change staging/unstaging controls; summary commit viewer

**Testing**
- Test navigator rewritten in Swift: 45% faster when running/reporting results; organized around test plans; filter by result type (including expected failure)
- Test reports: high-level summary + Insights (pattern-based failure analysis); test list with per-device/configuration breakdown; performance metrics tab
- Automation explorer **[NEW]**: interactive video playback of UI tests; scrub timeline; touch/mouse event overlay; UI hierarchy inspection at point of failure

**Debugging**
- Full OSLog integration **[NEW]** in debug console: filter by subsystem, category, severity; structured presentation with severity background colors; hidden metadata available on demand; jump from log entry to source line
- `Logger` from OSLog framework for structured logging with privacy annotations

**Distribution**
- Xcode Cloud: TestFlight test notes in source (automatically attached to builds); notarization post-action (auto-notarizes and staples Mac apps)
- XCFramework signature verification **[NEW]**: framework inspector shows signer identity; warning when identity changes on update
- Privacy manifests **[NEW]**: bundled in frameworks and apps; Xcode aggregates all manifests into a privacy report; aids App Store Connect privacy nutrition label
- TestFlight internal testing distribution option **[NEW]**: team-only distribution, never released to customers
- Simplified distribution: bundled presets with recommended settings; one-click Distribute; desktop notifications for build status

**Simulator Downloads**
- All simulators (including iOS and visionOS) now optional downloads — Xcode ships smaller; choose simulators at install time

## APIs & Frameworks

**Swift (language/macros)**
- `#Preview` macro **[NEW]** — SwiftUI view, UIViewController, AppKit view controller, widget previews
- Swift Macro system (see session 10164): `@freestanding`, `@attached` macros; `#externalMacro`
- `@CaseDetection` example macro (MemberMacro protocol)

**OSLog**
- `Logger(subsystem:category:)` (existing, now surfaced in Xcode 15 console with full filtering)
- `Logger.info(_:)`, `.error(_:)`, `.notice(_:)`, `.debug(_:)` — severity levels reflected in console UI

**String Catalogs**
- `.xcstrings` file format **[NEW]**
- `LocalizedStringKey` / `String(localized:)` (existing, String Catalogs are the new backing format)
- `Edit > Convert to String Catalog` — migration action **[NEW]**

**Asset Catalogs**
- Swift symbol generation for color and image assets **[NEW]**
- `Color(_:)` / `Image(_:)` can now use generated symbol names instead of string literals

**Privacy Manifests**
- `PrivacyInfo.xcprivacy` file **[NEW]**
- Xcode privacy report aggregation **[NEW]**

**XCFramework**
- Digital signature verification in framework inspector **[NEW]**

## Code Highlights

New `#Preview` macro:
```swift
#Preview {
    AppDetailColumn(screen: .account)
        .backyardBirdsDataContainer()
}

#Preview("Placeholder View") {
    AppDetailColumn()
        .backyardBirdsDataContainer()
}
```

UIViewController preview:
```swift
#Preview {
    let controller = DetailedMapViewController()
    return controller
}
```

OSLog structured logging:
```swift
import OSLog
let logger = Logger(subsystem: "BackyardBirdsData", category: "Account")

func login(password: String) -> Error? {
    logger.info("Logging in user '\(username)'...")
    if let error {
        logger.error("User '\(username)' failed to log in. Error: \(error)")
    } else {
        logger.notice("User '\(username)' logged in successfully.")
    }
    return error
}
```

## Takeaways
- Migrate to String Catalogs now — the single-file format with build-time synchronization eliminates the manual `.strings` maintenance that causes translation drift.
- Asset catalog Swift symbol generation catches name typos at compile time rather than crashing at runtime.
- Use the Bookmarks navigator to track multi-file refactoring tasks and in-progress work — it replaces ad-hoc `TODO:` comments and Find-in-Project searches.
- Add a `PrivacyInfo.xcprivacy` manifest to your SDKs and apps early; Apple will require it for privacy-impacting frameworks.

---
_Source: WWDC23 Session 10165 page (abstract, chapter summaries, code samples, and resource links)._
