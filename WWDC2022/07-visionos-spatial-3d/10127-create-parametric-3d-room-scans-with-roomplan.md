# Create Parametric 3D Room Scans with RoomPlan
**WWDC22 · Session 10127** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10127/)

_Platforms:_ iOS 16, iPadOS 16 (LiDAR-enabled iPhone and iPad Pro)

## Overview
RoomPlan is a brand-new framework announced at WWDC22 that uses the LiDAR scanner, ARKit, and on-device machine learning to create parametric 3D models of indoor rooms. A scan detects walls, windows, openings, and doors (as 2D `Surface` structures) and furniture objects like sofas, tables, chairs, and beds (as 3D `Object` structures), all packed into a fully parametric `CapturedRoom` data structure that can be exported to USD/USDZ.

The framework provides two integration points: `RoomCaptureView`, a drop-in `UIView` subclass with built-in ARKit visualization, real-time 3D model overlay, and coaching instructions; and the lower-level `RoomCaptureSession` + `RoomBuilder` data API for apps that need custom visualization or programmatic access to the live scan data. Both paths produce the same `CapturedRoom` output.

## Key Topics

### RoomCaptureView — Drop-in Scanning UI
`RoomCaptureView` is a `UIView` subclass that handles the complete scanning experience: animated wall/object outlines in world space, a live 3D model thumbnail, and text coaching. To integrate, create a `RoomCaptureView`, call `captureSession.run(configuration:)` to start, and `captureSession.stop()` to finish. Optionally conform to `RoomCaptureViewDelegate` to intercept or receive the final `CapturedRoom`.

### RoomCaptureSession — Data API
For custom visualizations, use `RoomCaptureSession` directly. It exposes the underlying `ARSession` for rendering in an `ARView`. Conform to `RoomCaptureSessionDelegate` to receive:
- `captureSession(_:didUpdate:)` — live `CapturedRoom` updates for real-time AR visualization
- `captureSession(_:didProvide:)` — `Instruction` values for user coaching (distance, speed, lighting, texture)
- `captureSession(_:didEndWith:error:)` — `CapturedRoomData` when the session ends

### RoomBuilder — Post-Processing
`RoomBuilder` processes `CapturedRoomData` into a final polished `CapturedRoom` using async/await. The `.beautifyObjects` option improves object geometry. Processing completes within a few seconds.

### CapturedRoom Data Model
- `walls`, `doors`, `windows`, `openings` — arrays of `CapturedRoom.Surface` (2D parametric planar elements)
- `objects` — array of `CapturedRoom.Object` (3D cuboid furniture items)
- `Surface` properties: `dimensions`, `confidence` (`.low`/`.medium`/`.high`), `transform`, `identifier`, `edges` (four edges), curve properties (`radius`, `startAngle`, `endAngle`), `category` (`.wall`, `.opening`, `.window`, `.door`)
- `Object` properties: `dimensions`, `confidence`, `transform`, `identifier`, `category` (furniture types)
- `CapturedRoom.export(to:)` — exports USD/USDZ file

### Best Practices
- Rooms up to 30×30 ft (9×9 m); minimum 50 lux lighting
- Prepare room: open curtains, close doors, remove mirror obstructions
- Avoid scans over 5 minutes to manage thermals
- Full-height mirrors, glass, dark surfaces, and very high ceilings can reduce accuracy

## APIs & Frameworks

### RoomPlan — RoomCaptureView
- `RoomCaptureView: UIView` **[NEW]** — complete scanning UI; access via `.captureSession`
- `RoomCaptureSession.Configuration` **[NEW]** — session configuration object
- `RoomCaptureSession.run(configuration:)` **[NEW]** — starts the scan
- `RoomCaptureSession.stop()` **[NEW]** — stops the scan
- `RoomCaptureViewDelegate` protocol **[NEW]**
  - `captureView(shouldPresent:error:) -> Bool` — opt out of post-processed result presentation
  - `captureView(didPresent:error:)` — receive final `CapturedRoom`

### RoomPlan — RoomCaptureSession (Data API)
- `RoomCaptureSession` **[NEW]** — low-level session; `arSession: ARSession` property for ARKit integration
- `RoomCaptureSessionDelegate` protocol **[NEW]**
  - `captureSession(_:didUpdate:)` — live `CapturedRoom` updates
  - `captureSession(_:didProvide:)` — `RoomCaptureSession.Instruction` coaching hints
  - `captureSession(_:didEndWith:error:)` — final `CapturedRoomData`
- `RoomCaptureSession.Instruction` **[NEW]** — enum: `.moveCloserToWall`, `.moveAwayFromWall`, `.slowDown`, `.turnOnLight`, `.normal`, `.lowTexture`

### RoomPlan — RoomBuilder
- `RoomBuilder` **[NEW]** — post-processes `CapturedRoomData` into `CapturedRoom`
- `RoomBuilder(options:)` — options: `.beautifyObjects`
- `RoomBuilder.capturedRoom(from:) async throws -> CapturedRoom` **[NEW]** — async processing

### RoomPlan — CapturedRoom
- `CapturedRoom: Codable, Sendable` **[NEW]** — root parametric room structure
- `CapturedRoom.Surface` **[NEW]** — wall/door/window/opening; `category`, `dimensions`, `transform`, `confidence`, `identifier`, `edges`, `curve` properties
- `CapturedRoom.Object` **[NEW]** — furniture cuboid; `category` (`CapturedRoom.Object.Category`), `dimensions`, `transform`, `confidence`, `identifier`
- `CapturedRoom.Object.Category` **[NEW]** — `.bathtub`, `.bed`, `.chair`, `.dishwasher`, `.fireplace`, `.oven`, `.refrigerator`, `.screen`, `.sink`, `.sofa`, `.stairs`, `.storage`, `.stove`, `.table`, `.toilet`, `.washerDryer`
- `CapturedRoom.export(to: URL) throws` **[NEW]** — exports USD or USDZ
- `CapturedRoomData` **[NEW]** — raw scan data; input to `RoomBuilder`

## Code Highlights

Using `RoomCaptureView` (simplest integration):
```swift
import RoomPlan
class RoomCaptureViewController: UIViewController {
    var roomCaptureView: RoomCaptureView!
    let config = RoomCaptureSession.Configuration()

    func startSession() { roomCaptureView.captureSession.run(configuration: config) }
    func stopSession()  { roomCaptureView.captureSession.stop() }
}
extension RoomCaptureViewController: RoomCaptureViewDelegate {
    func captureView(didPresent result: CapturedRoom, error: Error?) {
        try? result.export(to: destinationURL)
    }
}
```

Post-processing with `RoomBuilder`:
```swift
var roomBuilder = RoomBuilder(options: [.beautifyObjects])
func captureSession(_ session: RoomCaptureSession, didEndWith data: CapturedRoomData, error: Error?) {
    Task {
        let finalRoom = try await roomBuilder.capturedRoom(from: data)
        previewVisualizer.update(model: finalRoom)
    }
}
```

## Takeaways
- RoomPlan is a new, fully integrated framework that delivers parametric room scanning in a few lines of code — no custom ML or computer vision required.
- `RoomCaptureView` is the fastest path to integration; `RoomCaptureSession` + `RoomBuilder` give full control for custom AR visualizations.
- The `CapturedRoom` output is fully parametric (dimensions, transforms, confidence levels) and exportable as USD/USDZ for use in design, real estate, and e-commerce workflows.
- Requires a LiDAR-enabled device (iPhone Pro, iPad Pro); best results in well-lit rooms under 30×30 ft with scans kept under 5 minutes.

---
_Source: WWDC22 Session 10127 page (abstract, chapter summaries, code samples, and resource links)._
