# Use foveated streaming to bring immersive content to visionOS
**WWDC26 · Session 286** · [Watch](https://developer.apple.com/videos/play/wwdc2026/286/)

_Platforms:_ visionOS 27 (receiver app), macOS / Windows (streaming endpoint)

## Overview
The Foveated Streaming framework (new in visionOS 26.4) lets existing macOS and PC OpenXR applications stream immersive content wirelessly to Apple Vision Pro at full visual fidelity. Eye-tracked foveated video compression allocates encoding bits where the user is actually looking, delivering high-quality visuals at practical Wi-Fi bandwidth while protecting user privacy (gaze data never leaves the device).

The session covers the full integration on both sides: a visionOS receiver app built with `FoveatedStreamingSession`, and a Windows/Mac streaming endpoint implementing the Foveated Streaming Protocol via NVIDIA CloudXR. The architecture is open: the Foveated Streaming Protocol spec is published, the sample receiver and OpenXR client are on GitHub, and any streaming technology that implements the protocol can be used.

The session covers performance measurement (a dedicated Xcode instrument), enhancement patterns (message channels for data exchange, ARKit for real-world anchoring, RealityKit for compositing native 3D content alongside the stream), and privacy: gaze data used for foveation never leaves Apple Vision Pro.

## Key Topics

### Architecture
- **visionOS receiver app**: uses `FoveatedStreamingSession` to connect, pair, and display streamed content.
- **Streaming endpoint**: macOS or Windows machine running an OpenXR app with the Foveated Streaming Protocol; NVIDIA CloudXR is the reference implementation.
- `ImmersiveSpace(foveatedStreaming:)` SwiftUI scene modifier renders the stream in a full immersive space.
- Native SwiftUI windows and RealityKit content can be composited on top of the stream.

### Setting Up the Streaming Endpoint
- Open-source sample code on GitHub provides a reference Foveated Streaming Protocol implementation and an example OpenXR application.
- Authentication and pairing are handled by the protocol; the OpenXR runtime passes input data and depth buffers.

### Creating the visionOS Receiver App
- `FoveatedStreamingSession()` — create once as an app-level object.
- `session.connect()` async — initiates connection.
- `ImmersiveSpace(foveatedStreaming: session)` — displays the stream; replaces `@main` app scene.
- Add native SwiftUI windows with `Window(...)` alongside the `ImmersiveSpace`.
- Add RealityKit overlays with `RealityView { ... }` inside the `ImmersiveSpace`.
- `SpatialContainer` — positions SwiftUI views in the streamed space.

### Performance Measurement
- **Foveated Streaming instrument** in Xcode **[NEW]**: measures stream bandwidth, pose latency, and frame rate.
- Use to diagnose network bottlenecks and rendering lag before shipping.

### Enhancing with visionOS Features
- **Message channels**: bidirectional data exchange between the visionOS app and the OpenXR streaming endpoint (e.g., sending controller state, receiving game events).
- **ARKit**: anchor virtual content to real-world surfaces or objects in the mixed reality passthrough.
- **RealityKit**: composite native 3D objects (portals, notifications, UI panels) alongside the streamed frame.
- Alpha channel compositing: the streaming endpoint can render transparent regions to let passthrough show through.

### Privacy
- Eye tracking data used for foveated compression never leaves Apple Vision Pro — the endpoint only receives the compressed video frame, not gaze coordinates.

## APIs & Frameworks

### FoveatedStreaming (NEW framework — visionOS 26.4+)
- `FoveatedStreamingSession` **[NEW]**: core session object
  - `connect()` async throws
- `ImmersiveSpace(foveatedStreaming:)` **[NEW]**: SwiftUI scene modifier
- `ImmersiveSpace(foveatedStreaming:) { RealityView { ... } }` — compose RealityKit content **[NEW]**
- `SpatialContainer { ... }` **[NEW]**: positions SwiftUI views within streamed immersive space

### SwiftUI (visionOS)
- `@main struct App: App` with `Window(...)` and `ImmersiveSpace(foveatedStreaming:)` scenes

### RealityKit (composited with stream)
- `RealityView` — existing; nested inside `ImmersiveSpace(foveatedStreaming:)` for native 3D overlays

### ARKit (used for anchoring)
- World tracking, plane detection, and anchor APIs — existing; usable alongside streamed content

### Xcode Instruments
- **Foveated Streaming instrument** **[NEW]**: bandwidth, pose latency, frame rate metrics

### External Dependencies
- NVIDIA CloudXR SDK (Windows/macOS): OpenXR runtime for streaming endpoint
- Foveated Streaming Protocol spec (published by Apple): open standard for endpoint integration
- GitHub sample: `apple/StreamingSession` — reference receiver and OpenXR client

### Documentation
- [Foveated Streaming](https://developer.apple.com/documentation/FoveatedStreaming)
- [Creating a foveated streaming client on visionOS](https://developer.apple.com/documentation/FoveatedStreaming/creating-a-foveated-streaming-client-on-visionos)
- [Establishing foveated streaming sessions with Apple Vision Pro](https://developer.apple.com/documentation/FoveatedStreaming/establishing-foveated-streaming-sessions-with-apple-vision-pro)
- [Streaming a CloudXR application to Apple Vision Pro with foveation](https://developer.apple.com/documentation/FoveatedStreaming/streaming-a-cloudxr-application-to-apple-vision-pro-with-foveation)
- [Analyzing the performance of a foveated streaming session](https://developer.apple.com/documentation/FoveatedStreaming/analyzing-the-performance-of-a-foveated-streaming-session)

## Code Highlights

Minimal receiver app:
```swift
@main struct FoveatedStreamingSampleApp: App {
    private let session = FoveatedStreamingSession()
    var body: some SwiftUI.Scene {
        ImmersiveSpace(foveatedStreaming: session)
    }
}
```

Connect on button tap:
```swift
Button("Connect") {
    Task { try await session.connect() }
}
```

Compose RealityKit content over the stream:
```swift
ImmersiveSpace(foveatedStreaming: session) {
    RealityView { content in /* add native 3D entities */ }
}
```

Add a SwiftUI window alongside the stream:
```swift
Window("Main", id: appModel.mainWindowId) {
    ContentView(session: session)
}
ImmersiveSpace(foveatedStreaming: session) {
    SpatialContainer {
        TransformStreamWidgetView().environment(session)
    }
}
```

## Takeaways
- Foveated Streaming enables developers to bring high-end PC and Mac OpenXR experiences to Apple Vision Pro without porting to native visionOS — a major unlock for GPU-intensive applications like professional 3D tools and AAA games.
- The privacy-preserving design (gaze data stays on device) addresses a key concern for enterprise and consumer deployments.
- Native visionOS features (ARKit, RealityKit, SwiftUI) compose cleanly alongside the stream, allowing a hybrid experience that combines the power of a remote renderer with the device integration of native APIs.
- The open Foveated Streaming Protocol means any streaming technology — not just NVIDIA CloudXR — can become a compatible endpoint.

---
_Source: WWDC26 Session 286 page (abstract, chapter summaries, code samples, and resource links)._
