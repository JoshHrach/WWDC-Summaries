# Keep Colors Consistent Across Captures
**WWDC24 · Session 10162** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10162/)

_Platforms:_ iOS 18, iPadOS 18 (iPhone 14 family, iPhone 15 family, 2024 iPad Pro)

## Overview
This session introduces the Constant Color API — a new AVFoundation capture mode that determines the color of objects, people, and materials independently of the ambient lighting conditions. Unlike normal iPhone photography (which intentionally captures the mood and warmth of ambient light), Constant Color images represent the intrinsic color of surfaces as if photographed in a standardized, controlled environment.

The technique uses the improved flash hardware introduced in iPhone 14, combined with precise factory calibration of each device's flash and camera, computational photography, and machine learning. It captures a flash/no-flash pair and analyzes the brightness increase to predict what a dark-room image under a standard D65 illuminant would look like. Validation showed an 87% reduction in color variance across different lighting conditions for the iPhone 15 family.

## Key Topics

**What Is Color Constancy?**
- Human vision automatically corrects for ambient light cast; cameras normally do not (and the aesthetic result is intentional)
- Constant Color provides a machine equivalent: capturing surfaces' true color regardless of illumination
- Use cases: product marketing photography (showing accurate colors to online shoppers), dermatology/medical skin tracking (bruises, rashes, wound progression)

**How Constant Color Works**
- Requires flash hardware (iPhone 14+): flash and camera properties are factory-measured per device
- Two images captured in rapid succession: flash image + no-flash (ambient) image
- Images are registered (aligned) to mitigate motion between frames
- In the linear scene-referred domain, images are normalized for relative exposure
- ML + computational photography predict the "dark room" equivalent image
- Output is rendered as a D65-illuminated image with global tone mapping and gamma 2.2 encoding (no local tone mapping, to preserve subtle lightness variation)
- Result: ambient-light shadows are removed; surface colors are consistent across indoor lighting temperatures (2800K–7500K, 10–800 Lux tested)

**Code Integration (Six Steps)**
1. Check `AVCapturePhotoOutput.isConstantColorSupported` — changes when switching cameras or formats; supported on iPhone 14/15, 2024 iPad Pro
2. Configure pipeline: `builtInWideAngleCamera` or `builtInDualWideCamera`; session preset accordingly; set `isConstantColorEnabled = true` on `AVCapturePhotoOutput`; do NOT use RAW format (`rawPhotoPixelFormatType = 0`)
3. Set flash mode to `.auto` or `.on` (`.off` throws an exception for Constant Color)
4. Set `AVCapturePhotoSettings.isConstantColorEnabled = true` per capture
5. Receive a `CVPixelBuffer` confidence map alongside the image: values 0.0–1.0 per region (1.0 = full confidence, 0.0 = no confidence); suggested threshold 0.8–0.9
6. Use `constantColorCenterWeightedMeanConfidenceLevel` for a single summary value (center-weighted average); opt in to `isConstantColorFallbackPhotoDeliveryEnabled` to receive the ambient fallback photo if confidence is too low

**Confidence Map Details**
- Low confidence regions occur when: the flash doesn't brighten background sufficiently (distant scenes, outdoor daylight), or specular reflections clip pixel values
- Even low-confidence regions remain visually pleasing — the API fills in using flash data rather than leaving artifacts
- `didFinishProcessingPhoto` fires twice when fallback delivery is enabled: once for the Constant Color image, once for the ambient fallback (check `isConstantColorFallbackPhoto` property)
- Used in 2024 iPad Pro document scanning (shadow removal) at threshold 0.9

## APIs & Frameworks

**AVFoundation**
- `AVCapturePhotoOutput.isConstantColorSupported` **[NEW]** — check device support before enabling
- `AVCapturePhotoOutput.isConstantColorEnabled` **[NEW]** — must be set `true` to configure pipeline (triggers render pipeline reconfiguration; set once at startup)
- `AVCapturePhotoSettings.isConstantColorEnabled` **[NEW]** — request a Constant Color photo per capture
- `AVCapturePhotoSettings.isConstantColorFallbackPhotoDeliveryEnabled` **[NEW]** — opt in to receiving ambient fallback frame
- `AVCapturePhoto.constantColorConfidenceMap` **[NEW]** — `CVPixelBuffer` of Float values (0.0–1.0) per image region
- `AVCapturePhoto.constantColorCenterWeightedMeanConfidenceLevel` **[NEW]** — single Float summary statistic
- `AVCapturePhoto.isConstantColorFallbackPhoto` **[NEW]** — distinguishes Constant Color output from ambient fallback in `didFinishProcessingPhoto`
- `AVCapturePhotoCaptureDelegate.didFinishProcessingPhoto(_:error:)` — fires twice when fallback delivery is enabled
- `AVCaptureFlashMode.auto`, `.on` — required for Constant Color (`.off` disallowed)
- `CVPixelBuffer` — used to store/display both camera frames and confidence map
- `AVCaptureSession.sessionPreset` — must be configured appropriately for wide-angle camera
- `rawPhotoPixelFormatType = 0` — RAW not supported with Constant Color

## Code Highlights

Check support and enable in pipeline:
```swift
if photoOutput.isConstantColorSupported {
    photoOutput.isConstantColorEnabled = true
}
```

Request a Constant Color capture and handle confidence:
```swift
// In capture settings
settings.isConstantColorEnabled = true
settings.isConstantColorFallbackPhotoDeliveryEnabled = true

// In delegate
func photoOutput(_ output: AVCapturePhotoOutput,
                 didFinishProcessingPhoto photo: AVCapturePhoto, error: Error?) {
    if photo.isConstantColorFallbackPhoto {
        // Use ambient fallback
    } else {
        let confidence = photo.constantColorCenterWeightedMeanConfidenceLevel
        let map = photo.constantColorConfidenceMap
        // confidence >= 0.9 → use Constant Color image
    }
}
```

## Takeaways
- Add `isConstantColorSupported` / `isConstantColorEnabled` checks at app startup — the pipeline reconfiguration is expensive and should only happen once.
- Use the confidence map to determine if your specific region of interest (not just the center) has accurate colors; don't rely solely on the center-weighted summary.
- Set a threshold of 0.8–0.9 for general use; use 0.9 (as Apple does for document scanning) for high-accuracy applications.
- Shadow removal is a side effect of Constant Color — useful for document scanning, but be aware it changes the aesthetic of the image.

---
_Source: WWDC24 Session 10162 page (abstract, chapter summaries, code samples, and resource links)._
