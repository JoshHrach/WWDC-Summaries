# Explore Low-Latency Video Encoding with VideoToolbox
**WWDC21 · Session 10158** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10158/)

_Platforms:_ iOS 15, macOS Monterey 12

## Overview
This session introduces a new low-latency hardware H.264 encoding mode in VideoToolbox, activated via a single property in the encoder specification dictionary. The mode is designed for real-time video communication applications (video conferencing, live broadcasting) and optimizes the encoder pipeline in five areas: eliminating frame reordering for one-in/one-out encoding, faster rate-control adaptation to network congestion, temporal scalability for multi-recipient efficiency, max frame quantization parameter control for quality floors, and long-term reference (LTR) frames for network error resilience.

The feature is available on both iOS and macOS and always uses the hardware encoder for power efficiency. The new mode can reduce end-to-end encoding latency by up to 100 ms for 720p 30fps video.

## Key Topics
- **Low-Latency Mode Activation:** Set `kVTVideoEncoderSpecification_EnableLowLatencyRateControl = kCFBooleanTrue` in the `encoderSpecification` dictionary passed to `VTCompressionSessionCreate`. All other session creation and configuration APIs remain unchanged.
- **Latency Reduction Mechanisms:** Eliminates frame reordering (no B-frames); one frame in → one encoded frame out immediately. Rate controller adapts faster to network bandwidth changes. Net result: up to 100 ms lower latency than default mode for 720p 30fps.
- **New H.264 Profiles (NEW):** Two new profile level constants:
  - `kVTProfileLevel_H264_ConstrainedBaseline_AutoLevel` (CBP) – For low-cost decoders
  - `kVTProfileLevel_H264_ConstrainedHigh_AutoLevel` (CHP) – Better compression, requires capable decoders
- **Temporal Scalability (NEW):** `kVTCompressionPropertyKey_BaseLayerFrameRateFraction = 0.5` splits the bitstream into base layer (half the frames, fully decodable independently) and enhancement layer (remaining frames, supplemental). Base layer uses ~60% of the bit rate by default; configurable via `kVTCompressionPropertyKey_BaseLayerBitRateFraction` (recommended range 0.6–0.8). Check `kCMSampleAttachmentKey_IsDependedOnByOthers` to identify base layer frames. Enhancement layer frames have no inter-layer dependencies, so dropping them on lossy networks does not affect other frames.
- **Max Frame QP (NEW):** `kVTCompressionPropertyKey_MaxAllowedFrameQP` caps the quantization parameter (1–51) to enforce a minimum quality floor. The rate controller may drop frames rather than exceed the QP cap if the bit rate budget runs out. Useful for screen sharing content where sharpness matters more than frame rate.
- **Long-Term Reference (LTR) Frames (NEW):** Enable with `kVTCompressionPropertyKey_EnableLTR`. Encoder signals LTR frames via `kCMSampleAttachmentKey_RequireLTRAcknowledgementToken` in the output sample buffer. App sends the token to the receiver, receives acknowledgement, then reports acknowledged tokens to the encoder via `kVTEncodeFrameOptionKey_AcknowledgedLTRTokens` (array). To trigger an LTR-P refresh frame (much smaller than an I-frame), set `kVTEncodeFrameOptionKey_ForceLTRRefresh` on the next frame encode call. Falls back to a key frame if no acknowledged LTR is available.

## APIs & Frameworks

**VideoToolbox**
- `VTCompressionSessionCreate` – Existing API; pass `encoderSpecification` dict with low-latency flag
- `kVTVideoEncoderSpecification_EnableLowLatencyRateControl` **[NEW]** – Enables low-latency H.264 hardware encoding mode
- `kVTCompressionPropertyKey_ProfileLevel` – Set to new constants for profile selection:
  - `kVTProfileLevel_H264_ConstrainedBaseline_AutoLevel` **[NEW]** – Constrained Baseline Profile
  - `kVTProfileLevel_H264_ConstrainedHigh_AutoLevel` **[NEW]** – Constrained High Profile
- `kVTCompressionPropertyKey_BaseLayerFrameRateFraction` **[NEW]** – Fraction of frames in base layer (0.5 = half); enables temporal scalability
- `kVTCompressionPropertyKey_BaseLayerBitRateFraction` **[NEW]** – Fraction of target bit rate for base layer (default 0.6; recommended 0.6–0.8)
- `kCMSampleAttachmentKey_IsDependedOnByOthers` – `true` for base layer frames, `false` for enhancement layer frames (used to route frames to appropriate recipients)
- `kVTCompressionPropertyKey_MaxAllowedFrameQP` **[NEW]** – Maximum allowed frame quantization parameter (1–51); frames dropped rather than exceeding QP cap
- `kVTCompressionPropertyKey_EnableLTR` **[NEW]** – Enables long-term reference frame support
- `kCMSampleAttachmentKey_RequireLTRAcknowledgementToken` **[NEW]** – Token in output sample buffer attachment; app must get acknowledgement from receiver
- `kVTEncodeFrameOptionKey_AcknowledgedLTRTokens` **[NEW]** – Array of acknowledged LTR tokens; passed as frame option to next encode call
- `kVTEncodeFrameOptionKey_ForceLTRRefresh` **[NEW]** – Triggers LTR-P refresh frame on next encode; falls back to key frame if no acknowledged LTR available
- `kVTCompressionPropertyKey_AverageBitRate` – Existing; sets target bit rate
- `VTSessionSetProperty` – Existing; used to configure all above properties
- `VTCompressionSessionEncodeFrame` – Existing; pass frame options dict with LTR properties

## Code Highlights
Creating a low-latency compression session:
```c
CFMutableDictionaryRef encoderSpecification =
    CFDictionaryCreateMutable(kCFAllocatorDefault, 0, NULL, NULL);
CFDictionarySetValue(encoderSpecification,
                     kVTVideoEncoderSpecification_EnableLowLatencyRateControl,
                     kCFBooleanTrue);

VTCompressionSessionRef compressionSession;
OSStatus err = VTCompressionSessionCreate(kCFAllocatorDefault,
                                           width, height,
                                           kCMVideoCodecType_H264,
                                           encoderSpecification,
                                           NULL, NULL,
                                           outputHandler,
                                           NULL,
                                           &compressionSession);
```

Enabling constrained profiles:
```c
// Constrained Baseline Profile (low-cost decoder compatibility)
VTSessionSetProperty(compressionSession,
                     kVTCompressionPropertyKey_ProfileLevel,
                     kVTProfileLevel_H264_ConstrainedBaseline_AutoLevel);

// Constrained High Profile (better compression)
VTSessionSetProperty(compressionSession,
                     kVTCompressionPropertyKey_ProfileLevel,
                     kVTProfileLevel_H264_ConstrainedHigh_AutoLevel);
```

Enabling temporal scalability:
```c
// Half of frames go to base layer
VTSessionSetProperty(compressionSession,
                     kVTCompressionPropertyKey_BaseLayerFrameRateFraction,
                     (__bridge CFTypeRef)@(0.5));
// Base layer gets 60% of bit rate
VTSessionSetProperty(compressionSession,
                     kVTCompressionPropertyKey_BaseLayerBitRateFraction,
                     (__bridge CFTypeRef)@(0.6));
```

Triggering an LTR-P refresh frame:
```c
CFMutableDictionaryRef frameOptions = CFDictionaryCreateMutable(...);
CFDictionarySetValue(frameOptions, kVTEncodeFrameOptionKey_ForceLTRRefresh, kCFBooleanTrue);
// Report previously acknowledged tokens:
CFDictionarySetValue(frameOptions, kVTEncodeFrameOptionKey_AcknowledgedLTRTokens, acknowledgedTokensArray);
VTCompressionSessionEncodeFrame(compressionSession, imageBuffer, pts, duration, frameOptions, NULL, NULL);
```

## Takeaways
- Enabling low-latency mode requires a single flag in the encoder specification; all other VideoToolbox APIs remain unchanged, making it straightforward to adopt in existing pipelines.
- Temporal scalability is the most impactful feature for multi-party video calls: one encode → multiple output qualities, reducing CPU/GPU load linearly with participant count.
- LTR frames provide a smaller, faster refresh mechanism than key frames on lossy networks, but require the application layer to implement acknowledgement signaling (e.g., via RTCP RPSI or a custom protocol).
- `MaxAllowedFrameQP` is specifically useful for screen sharing content where users are more sensitive to blurry text and UI than to reduced frame rate.

---
_Source: WWDC21 Session 10158 page (abstract, transcript, and code samples)._
