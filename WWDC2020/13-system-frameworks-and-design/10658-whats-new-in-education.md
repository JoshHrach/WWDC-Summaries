# What's New in Education
**WWDC20 · Session 10658** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10658/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
This session is an umbrella overview of Apple's education technology ecosystem in 2020, covering every layer from device management to student apps. The key developer-facing story is a set of interconnected changes: Schoolwork 2.0 gets a redesigned UI with richer `CLSContext` metadata (thumbnails, summaries, suggested age ranges) and a Handout Library; the new ClassKit Catalog API allows apps to publish their activity hierarchies as a web-accessible catalog so teachers can browse content without needing to have launched the app first; the Automatic Assessment Configuration framework brings standardized testing lock-down mode to macOS Big Sur (and iOS 14 via a new unified API); and Shared iPad gains a Temporary Session mode for credential-free use. Apple School Manager, Classroom, and Schoolwork also become more tightly integrated—classes created in any one of the three systems now sync automatically to the others.

The session explicitly points developers to companion sessions for deeper dives: "What's New in ClassKit" (10672), "What's New in Assessment" (10005), and "What's New in Managing Apple Devices."

## Key Topics
- **Schoolwork 2.0** — new design with Handout Library, Handout detail view showing per-student progress; App Activity Chooser now displays thumbnails and summary from `CLSContext` metadata **[NEW]**
- **ClassKit** — `CLSContext` metadata additions (thumbnail, summary, suggested age/completion time, progress reporting capabilities); `isAssignable` property; available on macOS **[NEW]**
- **ClassKit Catalog API** — REST web service; publish app activity hierarchy publicly; teacher discovery without prior app use; development and production environments **[NEW]**
- **Automatic Assessment Configuration framework** — `AEAssessmentSession` / `AEAssessmentConfiguration`; macOS Big Sur support; iOS UIKit assessment API deprecated **[NEW on macOS]**
- **Shared iPad — Temporary Session** — no Apple ID required; all session data deleted on sign-out; MDM supervised restriction to opt out **[NEW]**
- **Apple School Manager + Classroom + Schoolwork integration** — classes sync automatically across all three; teachers can create classes in Classroom that appear in Apple School Manager and Schoolwork **[NEW]**
- **Classroom** — pinch-to-zoom for screen viewing; AirPlay class invitation code **[NEW]**
- **Shared iPad best practices** — all user data must sync from the cloud; use long-lived `CKOperations` and `UIBackgroundTask` for background sync on sign-out

## APIs & Frameworks

**ClassKit**
- `CLSContext.thumbnail: CGImage?` — 330×330 px max; shown in Schoolwork's Activity Chooser **[NEW]**
- `CLSContext.summary: String?` — activity description shown in Schoolwork **[NEW]**
- `CLSContext.suggestedAge: NSRange` — age range in years **[NEW]**
- `CLSContext.suggestedCompletionTime: NSRange` — time in minutes **[NEW]**
- `CLSContext.isAssignable: Bool` — set `false` for container-only contexts **[NEW]**
- `CLSProgressReportingCapability` — declare progress types (percent, score, duration, etc.) **[NEW]**
- ClassKit now available on macOS (native and Catalyst) **[NEW]**

**ClassKit Catalog API (new web service)**
- `POST https://api.classkit-catalog.apple.com/v1/contexts` — publish context hierarchy **[NEW]**
- `POST .../v1/thumbnails` — upload activity thumbnails **[NEW]**
- JWT authentication via ClassKit Catalog API key from Apple Developer portal

**Automatic Assessment Configuration**
- `AEAssessmentSession` / `AEAssessmentConfiguration` — new framework replaces UIKit assessment API **[NEW on macOS]**
- `UIAccessibilityRequestGuidedAccessSession` — **[DEPRECATED]** on iOS; migrate to `AEAssessmentSession`
- `AEAssessmentConfiguration` — optional toggles for dictation, spell check, predictive keyboard **[NEW on iOS]**

**CloudKit / Background Tasks (Shared iPad)**
- `CKOperation` with `isLongLived = true` — continues after user logs out of Shared iPad session
- `UIBackgroundTaskIdentifier` / `UIApplication.beginBackgroundTask` — background sync on Shared iPad sign-out

**MDM (referenced)**
- New supervised restriction key — allows institutions to disable Temporary Session on Shared iPad **[NEW]**

## Code Highlights

No new code patterns beyond what is covered in sessions 10672 (ClassKit) and 10005 (Assessment). The key integration pattern for Shared iPad background sync:

```swift
// Begin a background task so cloud sync completes after user signs out of Shared iPad
var backgroundTask: UIBackgroundTaskIdentifier = .invalid
backgroundTask = UIApplication.shared.beginBackgroundTask(withName: "SyncOnSignOut") {
    // Expiration handler — end task to avoid termination
    UIApplication.shared.endBackgroundTask(backgroundTask)
    backgroundTask = .invalid
}

// Use a long-lived CKOperation so it survives session end
let operation = CKModifyRecordsOperation(recordsToSave: pendingRecords, recordIDsToDelete: nil)
operation.isLongLived = true
CKContainer.default().privateCloudDatabase.add(operation)
```

## Takeaways
- Adopt all new `CLSContext` metadata properties (thumbnail, summary, suggested age/completion time, progress reporting capabilities) immediately—Schoolwork 2.0 surfaces them in the Activity Chooser and the teacher will see them before making any assignment.
- Submit app activities to the ClassKit Catalog API so teachers can discover and assign content without having previously launched or navigated the app on their device.
- Migrate iOS assessment code from the deprecated `UIAccessibilityRequestGuidedAccessSession` to the new `AEAssessmentSession` API; the same code then works on macOS Big Sur via Mac Catalyst with no additional changes.
- All Shared iPad apps must sync user data to and from the cloud; at sign-out, use a long-lived `CKOperation` combined with a `UIBackgroundTask` to ensure pending writes complete before the session is cleared.

---
_Source: WWDC20 Session 10658 page (transcript and resource links)._
