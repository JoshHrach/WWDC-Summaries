# What's New in ClassKit
**WWDC20 · Session 10672** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10672/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11 (native and Catalyst)

## Overview
ClassKit receives two categories of updates in 2020. First, `CLSContext` gains a set of rich metadata properties—thumbnail, summary, suggested age range, suggested completion time, and progress reporting capabilities—that surface detailed activity information to teachers in the Schoolwork app before they make an assignment. A new `isAssignable` property allows container contexts (e.g., a whole book) to be excluded from the assignable activity list, and new context types `.course` and `.custom` (with a settable `customTypeName`) cover content categories not handled by previous enumeration cases. A read-only `identifierPath` property returns the full path of a context in the hierarchy.

Second, the entirely new ClassKit Catalog API (`api.classkit-catalog.apple.com/v1`) is a REST web service that decouples content management from app releases. Developers upload a JSON representation of their full `CLSContext` hierarchy—including all new metadata fields and a separate thumbnail endpoint—using JWT authentication from a ClassKit Catalog API key provisioned in the Apple Developer portal. The catalog then serves context data directly to Schoolwork on teachers' and students' devices. The Catalog API is intended for public, static content; dynamically generated or user-specific contexts continue to use the native `CLSContext` API.

## Key Topics
- **`CLSContext.thumbnail`** — `CGImage?`; max 330×330 px; ClassKit downsizes if needed **[NEW]**
- **`CLSContext.summary`** — `String?`; describes the activity to teachers **[NEW]**
- **`CLSContext.suggestedAge`** — `NSRange`; age range in years (e.g., 9–11) **[NEW]**
- **`CLSContext.suggestedCompletionTime`** — `NSRange`; time range in minutes **[NEW]**
- **`CLSProgressReportingCapability`** — describes one kind of progress data an activity reports **[NEW]**
  - `.percent`, `.duration`, `.score`, `.quantity`, `.binary` kinds
  - `details: String` — localized description shown in Schoolwork
  - `CLSContext.addProgressReportingCapabilities([...])` — register capabilities
- **`CLSContext.isAssignable`** — `Bool`; set `false` on container contexts (books, courses) that should not appear as standalone assignments **[NEW]**
- **`CLSContext.identifierPath`** — `[String]`; read-only full identifier path **[NEW]**
- **`CLSContextType.course`** and `.custom` — new context type cases **[NEW]**
- **`CLSContext.customTypeName`** — optional string for `.custom` type contexts **[NEW]**
- **ClassKit Catalog API** — REST service for uploading CLSContext hierarchy as JSON; development and production environments; JWT authentication **[NEW]**
- **macOS availability** — ClassKit now available for native macOS and Catalyst apps **[NEW]**

## APIs & Frameworks

**ClassKit (native)**
- `CLSContext` — updated with new properties:
  - `thumbnail: CGImage?` — 330×330 px max **[NEW]**
  - `summary: String?` — activity description **[NEW]**
  - `suggestedAge: NSRange` — e.g., `NSRange(9...11)` **[NEW]**
  - `suggestedCompletionTime: NSRange` — minutes, e.g., `NSRange(15...20)` **[NEW]**
  - `isAssignable: Bool` — default `true`; set `false` for container-only contexts **[NEW]**
  - `identifierPath: [String]` — read-only full path **[NEW]**
  - `customTypeName: String?` — name for `.custom` type **[NEW]**
  - `addProgressReportingCapabilities([CLSProgressReportingCapability])` **[NEW]**
- `CLSContextType` — new cases: `.course`, `.custom` **[NEW]**
- `CLSProgressReportingCapability` **[NEW]** — describes reported data
  - `init(kind: CLSProgressReportingCapabilityKind, details: String)`
  - `CLSProgressReportingCapabilityKind`: `.percent`, `.duration`, `.score`, `.quantity`, `.binary`

**ClassKit Catalog REST API (new web service)**
- Base URL: `https://api.classkit-catalog.apple.com/v1`
- `POST /contexts?environment=development|production` — upload/update contexts (JSON array, max 200 per request) **[NEW]**
- `POST /thumbnails?environment=…` — upload a PNG/JPEG thumbnail (330×330 px); referenced by `thumbnail` field in context JSON **[NEW]**
- Authentication: `Authorization: Bearer <JWT>` — JWT signed with ECDSA SHA-256 using ClassKit Catalog API key from Developer portal
- Context JSON fields: all `CLSContext` properties + `metadata` object (`locale`, `minimumBundleVersion`, `keywords`)
- Error responses: JSON body with `id`, `code`, `message`

**CoreGraphics (referenced)**
- `CGImageSourceCreateWithURL(_:_:)` + `CGImageSourceCreateThumbnailAtIndex(_:_:_:)` — memory-efficient thumbnail creation

## Code Highlights

Add rich metadata to a quiz context:
```swift
let quizContext = CLSContext(type: .quiz,
                              identifier: "science_investigation_quiz",
                              title: "Measurements Quiz")

quizContext.summary = "A short quiz testing scientific measurements and data analysis."
quizContext.suggestedAge = NSRange(9...11)
quizContext.suggestedCompletionTime = NSRange(15...20)

// Thumbnail (max 330×330 px)
if let imageURL = Bundle.main.resourceURL?.appendingPathComponent("measurements_quiz.jpg"),
   let thumbnail = thumbnailFromImage(atURL: imageURL) {
    quizContext.thumbnail = thumbnail
}

// Progress reporting capabilities (localize details at runtime)
let percentCap = CLSProgressReportingCapability(kind: .percent,
                                                 details: NSLocalizedString("Reports percentage of progress", comment: ""))
let hintsCap = CLSProgressReportingCapability(kind: .quantity,
                                               details: NSLocalizedString("Reports number of hints used", comment: ""))
quizContext.addProgressReportingCapabilities([percentCap, hintsCap])
```

Memory-efficient thumbnail helper:
```swift
func thumbnailFromImage(atURL url: URL) -> CGImage? {
    guard let imageSource = CGImageSourceCreateWithURL(url as CFURL, nil) else { return nil }
    let options: [String: Any] = [
        kCGImageSourceCreateThumbnailFromImageAlways as String: true,
        kCGImageSourceThumbnailMaxPixelSize as String: 330
    ]
    return CGImageSourceCreateThumbnailAtIndex(imageSource, 0, options as CFDictionary)
}
```

Mark a container context as non-assignable:
```swift
let bookContext = CLSContext(type: .book, identifier: "fun_with_science", title: "Fun with Science")
bookContext.isAssignable = false  // whole book is just a container, not an assignable activity
```

## Takeaways
- Add thumbnail, summary, `suggestedAge`, `suggestedCompletionTime`, and `CLSProgressReportingCapability` entries to every `CLSContext`; these fields are displayed to teachers in Schoolwork before they make an assignment and are the primary signal for content quality.
- Set `isAssignable = false` on container contexts (books, courses, topic collections) that are only used to group child activities; this prevents them from appearing as standalone assignments in Schoolwork.
- Use the ClassKit Catalog API for any public, static content hierarchy to improve discoverability and allow content updates without submitting a new app binary; continue using the native `CLSContext` API for user-specific or dynamically generated content.
- Always localize `CLSProgressReportingCapability.details` and context titles at runtime based on the current locale; these strings are visible to teachers in Schoolwork.

---
_Source: WWDC20 Session 10672 page (transcript and code samples)._
