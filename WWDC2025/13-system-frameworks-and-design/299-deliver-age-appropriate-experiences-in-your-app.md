# Deliver Age-Appropriate Experiences in Your App
**WWDC25 · Session 299** · [Watch](https://developer.apple.com/videos/play/wwdc2025/299/)

_Platforms:_ iOS 26, iPadOS 26

## Overview
This session introduces the **Declared Age Range** framework — a new privacy-preserving API that lets apps request an age range from the user without ever seeing their birthdate. Parents or teens share only as much age information as the app needs (e.g., "is this user 16 or older?"), enabling age-gating and age-appropriate feature configuration while keeping exact birth dates private.

The session covers Apple's child safety roadmap context, how the framework works (range bucketing, caching, sharing settings), and a complete code walkthrough for requesting and responding to age range data.

## Key Topics

### Background: Child Safety Context
Apple's February 2025 white paper "Helping Protect Kids Online" preceded the framework. Additional 2025 changes: streamlined child account setup, parent-correctable child account ages, and new App Store global age ratings (4+, 9+, 13+, 16+, 18+). The Declared Age Range API is the developer-facing component of this initiative.

### How the API Works
Apps specify up to **3 age gate values** per request, resulting in 4 buckets. Each bucket must span at least 2 years. The API returns a range (lowerBound, upperBound — either may be nil for unbounded ranges like "12 or under" or "16 or over"). Birth dates are never exposed.

**Sharing modes** (parent-configurable):
- **Always Share** — automatically returns the age range; notifies on new information
- **Ask First** — prompts user each request cycle (default: once per year on anniversary)
- **Never Share** — silently declines

**Caching:** Responses are cached and synced across devices. Anniversary-based re-prompting prevents frequent age revelation. Users can clear the cache manually in Settings > Age Range for Apps.

**Declaration source:** `ageRangeDeclaration` on the response — `.guardianDeclared` for children/teens in Family Sharing, `.selfDeclared` for adults or teens not in Family.

### Implementation
1. Add "Declared Age Range" capability in Xcode (Signing & Capabilities tab).
2. Use `@Environment(\.requestAgeRange)` to get the request function.
3. Call `requestAgeRange(ageGates: ...)` with up to 3 age values.
4. Handle `.sharing(range:)` and `.declinedSharing` cases.
5. Check `range.lowerBound` and `range.upperBound` to determine feature eligibility.
6. Access `range.activeParentalControls` to respect settings like `.communicationLimits`.

### Related APIs
- **Sensitive Content Analysis API** — detects nudity in images, video; expanded in iOS 26 to live video streams.
- **Screen Time / Family Controls** — web usage supervision and parental controls.
- **PermissionKit** — communication limits for third-party apps (see session 293).

## APIs & Frameworks

**DeclaredAgeRange (iOS 26, iPadOS 26)**
- **[NEW]** `DeclaredAgeRange` framework — import for age-gating
- **[NEW]** `requestAgeRange(ageGates:)` — async function via SwiftUI environment; takes up to 3 age values
- **[NEW]** `@Environment(\.requestAgeRange)` — access the request function in SwiftUI
- **[NEW]** `AgeRangeResponse` — `.sharing(AgeRange)`, `.declinedSharing`
- **[NEW]** `AgeRange.lowerBound: Int?` — lower end of range (nil = unbounded low)
- **[NEW]** `AgeRange.upperBound: Int?` — upper end of range (nil = unbounded high)
- **[NEW]** `AgeRange.ageRangeDeclaration` — `.guardianDeclared`, `.selfDeclared`
- **[NEW]** `AgeRange.activeParentalControls` — set of active controls, e.g. `.communicationLimits`
- **[NEW]** `AgeRangeService.Error.invalidRequest` — malformed age gate request
- **[NEW]** `AgeRangeService.Error.notAvailable` — device not signed in to Apple Account

**Info.plist / Capabilities**
- "Declared Age Range" capability — required entitlement

## Code Highlights
Request age range in a SwiftUI view:
```swift
import SwiftUI
import DeclaredAgeRange

struct LandmarkDetail: View {
    @State var photoSharingEnabled = false
    @Environment(\.requestAgeRange) var requestAgeRange

    var body: some View {
        ScrollView { /* ... */ }
        .task { await requestAgeRangeHelper() }
    }

    func requestAgeRangeHelper() async {
        do {
            let response = try await requestAgeRange(ageGates: 16)
            switch response {
            case let .sharing(range):
                if let lower = range.lowerBound, lower >= 16 {
                    photoSharingEnabled = true
                }
                if range.activeParentalControls.contains(.communicationLimits) {
                    // Restrict communication features
                }
            case .declinedSharing:
                break
            }
        } catch AgeRangeService.Error.invalidRequest {
            print("Invalid request")
        } catch AgeRangeService.Error.notAvailable {
            print("Not available")
        }
    }
}
```

## Takeaways
- Use `requestAgeRange(ageGates:)` to gate features by age without seeing the user's actual birthday.
- Specify only the age boundaries your app actually cares about (up to 3); the API handles bucketing and privacy automatically.
- Always handle `.declinedSharing` gracefully — the default experience should not require age declaration.
- Check `range.activeParentalControls` to respect Communication Limits and other parental configurations alongside your own age-based logic.

---
_Source: WWDC25 Session 299 page (abstract, chapter summaries, code samples, and resource links)._
