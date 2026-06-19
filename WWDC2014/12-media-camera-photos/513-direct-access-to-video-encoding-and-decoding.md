# Direct Access to Video Encoding and Decoding
**WWDC14 · Session 513** · [Watch](https://developer.apple.com/videos/play/wwdc2014/513/)

_Platforms:_ iOS 8, OS X Yosemite 10.10

## Overview
This session covers how to leverage AVFoundation and Video Toolbox to gain direct, hardware-accelerated access to video encoding and decoding services on both iOS and OS X. The speaker walks through four concrete case studies—displaying a network H.264 stream in a layer, decoding to raw pixel buffers, compressing camera frames to a movie file, and compressing to compressed sample buffers for network transmission—showing which API level is appropriate for each scenario.

A second portion of the session introduces Multi-Pass Encoding, newly added to both AVFoundation and Video Toolbox in iOS 8 and OS X Yosemite. Multi-Pass improves quality-per-bit by allowing the encoder to analyze an entire sequence before making final bit-allocation decisions, using a frame silo and an encoder analysis database to support multiple passes over the source media.

The session emphasizes that hardware acceleration is available at every level of the stack (AVKit, AVFoundation, Video Toolbox) and that developers should choose the API level that matches their actual requirements rather than defaulting to low-level APIs unnecessarily.

## Key Topics

### Media Interface Stack
Overview of AVKit → AVFoundation → Video Toolbox → Core Media / Core Video layering. Hardware codecs are used at every level on iOS; on OS X they are used when available and appropriate.

### Core Media Types
Introduction to the fundamental types used across encoding/decoding APIs: `CVPixelBuffer`, `CVPixelBufferPool`, pixel buffer attributes dictionaries, `CMTime`, `CMVideoFormatDescription`, `CMBlockBuffer`, `CMSampleBuffer`, `CMClock`, and `CMTimebase`.

### Case Study 1 — AVSampleBufferDisplayLayer
Displaying a compressed H.264 network stream inside a CALayer. Covers converting Elementary Stream packaging (start-code NAL Units + in-band parameter sets) to MPEG-4 packaging (length-code NAL Units + parameter sets in `CMVideoFormatDescription`), building `CMSampleBuffer` objects, controlling display timing via a custom `CMTimebase`, and feeding buffers using both periodic and `requestMediaDataWhenReadyOnQueue` patterns.

### Case Study 2 — VTDecompressionSession
Decoding compressed frames to raw `CVPixelBuffer` objects for use in a custom render pipeline. Covers creating the session, specifying output pixel buffer attributes without over-constraining (to avoid unnecessary format-conversion copies), synchronous vs. asynchronous decode, back-pressure behavior, and handling `CMVideoFormatDescription` changes mid-stream.

### Case Study 3 — AVAssetWriter (referenced)
Compressing a CVPixelBuffer stream directly to a movie file via `AVAssetWriter`; detailed coverage deferred to prior WWDC sessions.

### Case Study 4 — VTCompressionSession
Compressing camera frames to `CMSampleBuffer` objects for direct network transmission. Covers session creation, key compression properties (`AllowFrameReordering`, `AverageBitRate`, `H264EntropyMode`, `RealTime`, `ProfileLevel`), feeding frames with `VTCompressionSessionEncodeFrame`, draining with `VTCompressionSessionCompleteFrames`, and converting MPEG-4 NAL Units back to Elementary Stream format for sending.

### Multi-Pass Encoding
Architecture of multi-pass (frame silo + encoder analysis database), hardware-acceleration parity with single-pass, AVFoundation API (`AVAssetExportSession.canPerformMultiplePassesOverSourceMediaData`, `AVAssetWriterInput` pass descriptions, `AVAssetReaderOutput` random access), and Video Toolbox API (`VTMultiPassStorage`, `VTFrameSilo`, `VTCompressionSessionBeginPass`, `VTCompressionSessionEndPass`, `VTCompressionSessionGetTimeRangesForNextPass`). Guidance on when multi-pass is and is not beneficial.

## APIs & Frameworks

**AVFoundation**
- `AVSampleBufferDisplayLayer` **[NEW on iOS 8]** — layer that decodes and displays CMSampleBuffers
  - `-enqueueSampleBuffer:`
  - `-requestMediaDataWhenReadyOnQueue:usingBlock:`
  - `-isReadyForMoreMediaData`
  - `.controlTimebase` (CMTimebase)
- `AVAssetWriter` / `AVAssetWriterInput`
  - `AVAssetWriterInput.canPerformMultiplePassesOverSourceMediaData` **[NEW]**
  - `-markCurrentPassAsFinished`
  - `-respondToEachPassDescriptionOnQueue:usingBlock:`
  - `.currentPassDescription` → `AVAssetWriterInputPassDescription`
  - `AVAssetWriterInputPassDescription.sourceTimeRanges`
- `AVAssetReaderOutput`
  - `.supportsRandomAccess` **[NEW]**
  - `-resetForReadingTimeRanges:` **[NEW]**
  - `-markConfigurationAsFinal` **[NEW]**
- `AVAssetExportSession`
  - `.canPerformMultiplePassesOverSourceMediaData` **[NEW]**

**Video Toolbox** (newly headered on iOS 8) **[NEW on iOS]**
- `VTDecompressionSession`
  - `VTDecompressionSessionCreate`
  - `VTDecompressionSessionDecodeFrame` (flags: `kVTDecodeFrame_EnableAsynchronousDecompression`)
  - `VTDecompressionSessionWaitForAsynchronousFrames`
  - `VTDecompressionSessionCanAcceptFormatDescription`
  - `VTDecompressionOutputCallback`
- `VTCompressionSession`
  - `VTCompressionSessionCreate`
  - `VTSessionSetProperty` — properties: `kVTCompressionPropertyKey_AllowFrameReordering`, `kVTCompressionPropertyKey_AverageBitRate`, `kVTCompressionPropertyKey_H264EntropyMode`, `kVTCompressionPropertyKey_RealTime`, `kVTCompressionPropertyKey_ProfileLevel`, `kVTCompressionPropertyKey_MultiPassStorage`
  - `VTCompressionSessionEncodeFrame`
  - `VTCompressionSessionCompleteFrames`
  - `VTCompressionSessionBeginPass` **[NEW]**
  - `VTCompressionSessionEndPass` **[NEW]**
  - `VTCompressionSessionGetTimeRangesForNextPass` **[NEW]**
  - `VTCompressionOutputCallback`
- `VTMultiPassStorage` **[NEW]**
  - `VTMultiPassStorageCreate`
  - `VTMultiPassStorageClose`
- `VTFrameSilo` **[NEW]**
  - `VTFrameSiloCreate`
  - `VTFrameSiloAddSampleBuffer`
  - `VTFrameSiloPrepareForReading`
  - `VTFrameSiloCopySampleBufferForTimeRange`

**Core Media**
- `CMVideoFormatDescriptionCreateFromH264ParameterSets` — builds format description from SPS/PPS NAL units
- `CMVideoFormatDescriptionGetH264ParameterSetAtIndex` — extracts parameter sets for Elementary Stream output
- `CMSampleBufferCreate`
- `CMBlockBufferCreate` / `CMBlockBufferCreateWithMemoryBlock`
- `CMTime` (64-bit value + 32-bit timescale)
- `CMClockGetHostTimeClock`
- `CMTimebaseCreate`, `CMTimebaseSetTime`, `CMTimebaseSetRate`

**Core Video**
- `CVPixelBuffer`, `CVPixelBufferPool`
- Pixel buffer attribute keys: `kCVPixelBufferOpenGLESCompatibilityKey`, `kCVPixelBufferWidthKey`, `kCVPixelBufferHeightKey`, `kCVPixelBufferPixelFormatTypeKey`

**Codec constants**
- `kCMVideoCodecType_H264`
- `kVTH264EntropyMode_CABAC`, `kVTH264EntropyMode_CAVLC`

## Code Highlights

Converting Elementary Stream H.264 to MPEG-4 packaging for CMSampleBuffer creation:
```objc
// Build CMVideoFormatDescription from SPS + PPS
CMVideoFormatDescriptionCreateFromH264ParameterSets(
    kCFAllocatorDefault,
    parameterSetCount,
    parameterSetPointers,
    parameterSetSizes,
    nalUnitHeaderLength,
    &formatDescription);

// Wrap length-prefixed NAL unit in CMBlockBuffer, then create sample buffer
CMSampleBufferCreate(kCFAllocatorDefault, blockBuffer, true,
    NULL, NULL, formatDescription,
    1, 1, &timing, 0, NULL, &sampleBuffer);
```

Feeding AVSampleBufferDisplayLayer with back-pressure:
```objc
[displayLayer requestMediaDataWhenReadyOnQueue:queue usingBlock:^{
    while (displayLayer.isReadyForMoreMediaData) {
        CMSampleBufferRef sb = [self nextSampleBuffer];
        if (!sb) break;
        [displayLayer enqueueSampleBuffer:sb];
    }
}];
```

Enabling multi-pass on AVAssetWriterInput:
```objc
writerInput.canPerformMultiplePassesOverSourceMediaData = YES;
[writerInput markCurrentPassAsFinished];
[writerInput respondToEachPassDescriptionOnQueue:queue usingBlock:^{
    AVAssetWriterInputPassDescription *pass = writerInput.currentPassDescription;
    if (pass) {
        [readerOutput resetForReadingTimeRanges:pass.sourceTimeRanges];
        [writerInput requestMediaDataWhenReadyOnQueue:queue usingBlock:feedBlock];
    } else {
        [writerInput markAsFinished];
    }
}];
```

## Takeaways

- Hardware-accelerated video encoding and decoding is available through AVFoundation as well as Video Toolbox; choose the highest-level API that meets your needs.
- Converting between Elementary Stream and MPEG-4 NAL Unit packaging is the key translation step required when building `CMSampleBuffer` objects from network streams.
- Avoid over-specifying pixel buffer attributes in `VTDecompressionSession`; unnecessary constraints force extra format-conversion copies that hurt performance and battery life.
- Multi-Pass Encoding delivers substantially better quality-per-bit for varying-complexity content (e.g., feature films, trailers) but is inappropriate for real-time encoding, power-constrained scenarios, or storage-constrained environments.

---
_Source: WWDC14 Session 513 page (abstract, chapter summaries, code samples, and resource links)._
