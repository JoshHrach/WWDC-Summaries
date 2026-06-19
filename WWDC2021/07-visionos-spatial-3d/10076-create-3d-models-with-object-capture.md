# Create 3D Models with Object Capture
**WWDC21 · Session 10076** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10076/)

_Platforms:_ macOS Monterey 12 (macOS 12+, Intel and Apple silicon)

## Overview
Object Capture is a new macOS API in RealityKit that uses photogrammetry to convert a folder of photos of a real-world object into a production-ready 3D USDZ model in minutes. Engineers take 20–200 overlapping photos of an object from all angles (using iPhone, iPad, DSLR, or drone), copy the images to a Mac, and then call `PhotogrammetrySession` to reconstruct the geometry, normals, and PBR material maps automatically.

The API is available on recent Intel Macs and runs significantly faster on Apple silicon by utilizing the Apple Neural Engine. It outputs USDZ files at four detail levels (Reduced, Medium, Full, Raw) optimized for different use cases — from AR Quick Look on iPhone to film-quality production pipelines. A companion iOS CaptureSample app (provided in developer documentation) captures depth-embedded HEIC images and gravity vectors to allow the API to automatically recover true object scale and upright orientation.

## Key Topics

### PhotogrammetrySession
`PhotogrammetrySession` (in `RealityKit`) is the primary API object. It accepts either a folder URL of images or a sequence of custom `PhotogrammetrySample` objects (for advanced pipelines that supply depth maps, gravity, or segmentation masks). Create the session once; it is a container for a fixed set of input images. Configuration parameters (default is sufficient for most cases) control advanced behavior.

### Async Output Stream
After creation, connect to `session.outputs` — a Swift `AsyncSequence` (new in Swift 5.5) — to receive output messages asynchronously. Message types include `.requestProgress` (fraction completed per request), `.requestComplete` (with the resulting model URL or entity), `.requestError`, `.processingComplete` (all queued requests done), and various warning types (e.g., unreadable images).

### Process Requests
Call `session.process(requests:)` with an array of `PhotogrammetrySession.Request` values. Submitting multiple detail levels in one call allows the engine to share computation, producing all models faster than sequential calls. Request types:
- `.modelFile(url:detail:geometry:)` — write a USDZ (or USDA/OBJ into a folder) to disk
- `.modelEntity(detail:geometry:)` — return a RealityKit `ModelEntity` for in-app display
- `.bounds` — return an estimated `BoundingBox` for the capture volume

### Detail Levels
| Level | Use Case | Material Channels |
|---|---|---|
| Reduced | Multiple objects in AR, web | Diffuse, Normal, AO |
| Medium | Single AR object, AR Quick Look | Diffuse, Normal, AO (more detail) |
| Full | Games, high-detail interactive | Diffuse, Normal, AO, Roughness, Displacement |
| Raw | Custom pro pipelines | Max poly + diffuse; no material baking |

### Interactive Workflow
For apps needing user control before final reconstruction: (1) request a `preview` detail model to display quickly, (2) request a `.bounds` BoundingBox, (3) let users adjust the capture volume and root transform in a 3D UI, (4) request a refined preview using the modified `geometry` parameter, then (5) produce final detail-level models.

### Capture Best Practices
- Objects should have adequate surface texture; avoid transparent or highly reflective surfaces.
- Diffuse/tent lighting eliminates hard shadows that confuse reconstruction.
- Move slowly, capturing uniform coverage; include flipped passes to reconstruct the bottom.
- Use the provided iOS `CaptureSample` app to embed stereo depth and gravity in HEIC files — enables automatic scale and orientation recovery.
- Turntable capture with a light tent yields the best results.

## APIs & Frameworks

**RealityKit** (`import RealityKit`) — **[NEW macOS 12]**

- `PhotogrammetrySession` **[NEW]** — primary reconstruction controller
  - `init(input: URL, configuration:)` — create session from image folder URL
  - `init(input: Sequence<PhotogrammetrySample>, configuration:)` — advanced sample sequence input
  - `outputs: AsyncThrowingStream<Output, Error>` **[NEW]** — async output message stream
  - `process(requests: [Request])` **[NEW]** — kick off one or more reconstruction requests
- `PhotogrammetrySession.Request` **[NEW]** — enum of request types
  - `.modelFile(url:detail:geometry:)` — write USDZ file to URL
  - `.modelEntity(detail:geometry:)` — reconstruct as `ModelEntity`
  - `.bounds` — estimate capture volume `BoundingBox`
- `PhotogrammetrySession.Output` **[NEW]** — enum of output messages
  - `.requestProgress(request:fractionComplete:)` — progress 0.0–1.0
  - `.requestComplete(request:result:)` — reconstruction finished; result is `.modelFile(URL)` or `.modelEntity(ModelEntity)` or `.boundingBox(BoundingBox)`
  - `.requestError(request:error:)` — reconstruction failed
  - `.processingComplete` — all queued requests done
- `PhotogrammetrySession.Configuration` **[NEW]** — advanced tuning parameters
- `PhotogrammetrySession.Request.Detail` **[NEW]** — `.preview`, `.reduced`, `.medium`, `.full`, `.raw`
- `PhotogrammetrySample` **[NEW]** — custom input: `image`, `depthDataMap`, `gravity`, `segmentationMask`

## Code Highlights

Creating a session from a folder of images:
```swift
import RealityKit

let inputFolderUrl = URL(fileURLWithPath: "/tmp/Sneakers/", isDirectory: true)
let session = try! PhotogrammetrySession(input: inputFolderUrl,
                                         configuration: PhotogrammetrySession.Configuration())
```

Connecting the async output stream:
```swift
Task {
    do {
        for try await output in session.outputs {
            switch output {
            case .requestProgress(let request, let fraction):
                print("Request progress: \(fraction)")
            case .requestComplete(let request, let result):
                if case .modelFile(let url) = result {
                    print("Request result output at \(url).")
                }
            case .requestError(let request, let error):
                print("Error: \(request) error=\(error)")
            case .processingComplete:
                print("Completed!")
                handleComplete()
            default:
                break
            }
        }
    } catch {
        print("Fatal session error! \(error)")
    }
}
```

Requesting two detail levels simultaneously:
```swift
try! session.process(requests: [
    .modelFile("/tmp/Outputs/model-reduced.usdz", detail: .reduced),
    .modelFile("/tmp/Outputs/model-medium.usdz", detail: .medium)
])
```

## Takeaways
- `PhotogrammetrySession` converts a folder of photos into a 3D USDZ model in minutes; requesting multiple detail levels in a single `process(requests:)` call shares computation for faster output.
- The Swift `AsyncSequence`-based `session.outputs` stream makes it straightforward to monitor progress and receive completed models without callbacks or polling.
- Depth-embedded HEIC images from a dual-camera iPhone or iPad automatically recover true object scale and upright orientation — no manual measurement required.
- Choose detail level by target platform: Reduced/Medium for AR Quick Look and web; Full for games; Raw for custom post-production pipelines.
- The interactive workflow (preview → adjust bounding box → refine → final) eliminates post-production edits by cropping geometry and optimizing the model before full reconstruction.

---
_Source: WWDC21 Session 10076 page (abstract, chapter summaries, code samples, and resource links)._
