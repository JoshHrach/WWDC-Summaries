# Enhance Your App with Machine Learning-Based Video Effects
**WWDC25 · Session 300** · [Watch](https://developer.apple.com/videos/play/wwdc2025/300/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26

## Overview
This session presents the Video Toolbox ML video processing pipeline, now expanded to iOS 26 from its macOS origins. The pipeline centers on `VTFrameProcessor` — a composable, protocol-based processing stage — and provides four production-ready ML effect implementations: frame rate conversion (motion-compensated), motion blur, super-resolution scaling, and frame interpolation. The session shows how to compose these effects in sequence and how to build a custom `VTFrameProcessor` for proprietary algorithms.

All effects run on the Neural Engine where available, making them practical for real-time playback, export, and live camera processing.

## Key Topics

### VTFrameProcessor Architecture
- `VTFrameProcessor` is the base protocol for any ML video processing stage. **[NEW on iOS 26]**
- Each processor has a **configuration** object (set at initialization, controls algorithmic parameters) and a **parameters** object (per-frame inputs such as source frame, timestamps, output pixel buffer).
- Processors are designed to be chained: the output of one feeds into the next.
- Multiple processors may share computation (e.g., an optical flow map computed once can be reused by both a frame-rate converter and a motion-blur stage).

### Frame Rate Conversion
- `VTFrameRateConversionConfiguration` — configures target frame rate and quality/speed trade-offs.
- `VTFrameRateConversionParameters` — per-frame inputs: source frame, output buffer, source/target timestamps.
- Uses motion-compensated interpolation; significantly higher quality than simple frame duplication or blending.
- Common use cases: upconvert 24 fps content to 60 fps for smooth playback; match display refresh rate.

### Motion Blur
- `VTMotionBlurConfiguration` — configures amount of blur and directional emphasis.
- `VTMotionBlurParameters` — per-frame inputs.
- Adds physically plausible motion blur to content that was rendered or captured without it (e.g., CGI, sports footage at very high shutter speeds).

### Super Resolution / Low-Latency Scaling
- `LowLatencySuperResolutionScalerConfiguration` — upscales a lower-resolution frame to a higher-resolution output using a learned model.
- Designed for real-time streaming and camera paths where latency budget is tight.

### Frame Interpolation
- `LowLatencyFrameInterpolationConfiguration` — generates intermediate frames to smooth playback at a fraction of the cost of a full render.
- Pairs naturally with super resolution: generate at a lower resolution and scale up, reducing both latency and bandwidth.

### Optical Flow
- `VTOpticalFlowConfiguration` — **[NEW]** computes per-pixel motion vectors between two frames.
- Can be shared across multiple downstream processors to avoid recomputation.
- Also useful as a standalone effect (e.g., custom motion-based segmentation).

### Building a Custom Processor
- Adopt `VTFrameProcessor` protocol; implement your own `Configuration` and `Parameters` types.
- The same composition and sharing model applies to custom processors — they integrate seamlessly into a `VTFrameProcessor` chain.

## APIs & Frameworks

### Video Toolbox (all NEW on iOS 26)
- `VTFrameProcessor` — base protocol for ML video processing stages.
- `VTFrameRateConversionConfiguration` / `VTFrameRateConversionParameters`
- `VTMotionBlurConfiguration` / `VTMotionBlurParameters`
- `LowLatencySuperResolutionScalerConfiguration`
- `LowLatencyFrameInterpolationConfiguration`
- `VTOpticalFlowConfiguration`
- Custom processor conformance via `VTFrameProcessor` protocol.

## Code Highlights

```swift
// Set up a frame rate conversion processor
let config = VTFrameRateConversionConfiguration(targetFrameRate: 60)
let processor = try VTFrameRateConversionProcessor(configuration: config)

// Process a frame
var params = VTFrameRateConversionParameters(
    sourceFrame: inputPixelBuffer,
    outputBuffer: outputPixelBuffer,
    sourceTimestamp: sourcePTS,
    targetTimestamp: targetPTS
)
try await processor.process(parameters: &params)
```

```swift
// Chained pipeline: optical flow → motion blur → super resolution
let flowConfig = VTOpticalFlowConfiguration()
let blurConfig = VTMotionBlurConfiguration(amount: 0.7)
let scalerConfig = LowLatencySuperResolutionScalerConfiguration(outputSize: targetSize)
let pipeline = try VTFrameProcessorPipeline(stages: [flowConfig, blurConfig, scalerConfig])
```

## Takeaways
- All four built-in effects are Neural Engine-accelerated and practical for real-time use on devices that support iOS 26.
- Start with `VTFrameRateConversionConfiguration` for frame rate upconversion — it delivers a significant quality improvement over simple interpolation with minimal integration effort.
- Share an optical flow stage across multiple downstream processors in a pipeline to avoid redundant computation.
- `VTFrameProcessor` is public and extensible — custom implementations integrate with the same API as Apple's built-in effects.

---
_Source: WWDC25 Session 300 page (abstract, chapter summaries, code samples, and resource links)._
