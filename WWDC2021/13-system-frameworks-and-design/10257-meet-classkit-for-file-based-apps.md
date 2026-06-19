# Meet ClassKit for File-Based Apps
**WWDC21 · Session 10257** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10257/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
ClassKit enables educational apps to surface student progress data to teachers via Apple's Schoolwork app. Previously, ClassKit's context-based API required apps to declare assignable content (`CLSContext`) up front, which did not fit the model of file-based apps that work with arbitrary documents. This session introduces a new `fetchActivity(for:)` API on `CLSDataStore` that directly ties progress reporting to a file URL, removing the need to pre-declare contexts.

The new file-based API is designed for any app that opens or edits files — text editors, document viewers, audio/video players, and more. When a teacher assigns a specific file to students via Schoolwork, the app can report duration, progress, and activity items (binary, quantity, or score) tied to that file, giving teachers rich insights into student engagement without the overhead of the context-based registration flow.

The session also walks through testing the ClassKit integration using Schoolwork's developer mode, which allows switching device roles between teacher and student without separate accounts.

## Key Topics

**New File-Based API**
`CLSDataStore.shared.fetchActivity(for: fileURL)` retrieves (or creates) a `CLSActivity` associated with a specific file URL. The app must support "Open in Place" so the same file instance is shared between student and teacher rather than a copy.

**Progress Data Types**
Four kinds of data can be attached to a `CLSActivity`:
1. **Duration** — `activity.start()` / `activity.stop()` track time-on-task
2. **Progress** — a `Double` from 0.0 to 1.0, or additive ranges via `addProgressRange(from:to:)`
3. **Primary Activity Item** — one highlighted `CLSActivityItem` shown in the main Schoolwork UI
4. **Additional Activity Items** — an array of supplementary `CLSActivityItem`s

**CLSActivityItem Subclasses**
- `CLSBinaryItem` — binary true/false data (e.g., correct/incorrect on a quiz question)
- `CLSQuantityItem` — any numeric value (e.g., word count, page count, slide count)
- `CLSScoreItem` — a score as part out of total (e.g., 8 out of 10)

**Testing with Developer Mode**
Set the ClassKit environment entitlement to `development` in Xcode, enable ClassKit API developer mode on-device in Settings > Developer, and switch roles between Teacher and Student to create and receive assignments without separate accounts. Revert to `production` before release.

## APIs & Frameworks

### ClassKit

**New File-Based API**
- `CLSDataStore.shared.fetchActivity(for: URL, completion: (CLSActivity?, Error?) -> Void)` — retrieves or creates activity for a file URL **[NEW]**
- `CLSDataStore.shared.fetchActivity(for: URL) async throws -> CLSActivity` — async/await variant **[NEW]**

**CLSActivity**
- `CLSActivity.start()` — begin tracking time
- `CLSActivity.stop()` — stop tracking time
- `CLSActivity.progress` — `Double` (0.0–1.0) progress value
- `CLSActivity.addProgressRange(from:to:)` — add a progress range (handles overlaps)
- `CLSActivity.primaryActivityItem` — `CLSActivityItem?` shown prominently in Schoolwork
- `CLSActivity.additionalActivityItems` — `[CLSActivityItem]`
- `CLSActivity.addAdditionalActivityItem(_:)` — append an additional item

**CLSActivityItem Subclasses**
- `CLSBinaryItem(identifier:title:type:)` — binary data item
- `CLSQuantityItem(identifier:title:)` — numeric quantity item
  - `.quantity` — `Double` value
- `CLSScoreItem(identifier:title:score:maxScore:)` — score item

**CLSDataStore**
- `CLSDataStore.shared.save()` — persist changes (must be called after every mutation)
- `CLSDataStore.shared.save() async throws` — async variant

**ClassKit Entitlement**
- `com.apple.developer.ClassKit-environment` — set to `development` for testing, `production` for release

## Code Highlights

Start tracking time when a file opens:
```swift
func openFile() async throws {
    let activity = try await CLSDataStore.shared.fetchActivity(for: fileURL)
    activity.start()
    try await CLSDataStore.shared.save()
}
```

Stop timer and record word count when file closes:
```swift
func closeFile() async throws {
    let activity = try await CLSDataStore.shared.fetchActivity(for: fileURL)
    let wordCount = activity.primaryActivityItem as? CLSQuantityItem ??
        CLSQuantityItem(identifier: "total_word_count", title: "Word Count")
    wordCount.quantity = currentDocumentWordCount()
    activity.primaryActivityItem = wordCount
    activity.progress = progress()
    activity.stop()
    try await CLSDataStore.shared.save()
}
```

## Takeaways
- The new `fetchActivity(for:)` API lets any file-based app report ClassKit progress data without pre-declaring `CLSContext` trees — just pass the file URL.
- Apps must support "Open in Place" (not copy-on-open) for the file-based ClassKit data to be correctly associated with the teacher's assigned file.
- Always call `CLSDataStore.shared.save()` after modifying an activity; unsaved changes are discarded.
- The ClassKit developer mode in Settings lets a single device simulate teacher and student roles for easy end-to-end testing.

---
_Source: WWDC21 Session 10257 page (abstract, chapter summaries, code samples, and resource links)._
