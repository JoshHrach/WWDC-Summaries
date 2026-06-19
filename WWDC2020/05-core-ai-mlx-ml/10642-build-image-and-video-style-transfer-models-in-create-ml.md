# Build Image and Video Style Transfer Models in Create ML
**WWDC20 · Session 10642** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10642/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
Style Transfer is a new machine learning task in the Create ML app on macOS Big Sur that lets developers train models to apply the visual style of one image (colors, textures, brush strokes, patterns) to any content image or video frame. Training takes only a few minutes in the Create ML app, produces models under 1 MB in size, and the resulting Core ML models are fast enough for real-time video processing on A13 Bionic at up to 120 frames per second.

The session demonstrates the full training workflow in the Create ML app — selecting a style image, adding content training images, configuring style strength and style density, monitoring training via loss graphs and live checkpoint previews, and exporting the model. A demo combining Style Transfer, ARKit, person segmentation, and Metal on an iPhone shows three Style Transfer models running concurrently in real time on a live AR scene.

## Key Topics

**Style Transfer Template in Create ML**
A new template in the Create ML app. Training requires: a single style image (the artistic source), an optional validation image (for live checkpoint visualization during training), and a directory of content images (hundreds of natural images; can be downloaded directly from the app). The model learns to balance style and content from these inputs.

**Optimization Target: Image vs. Video**
The training mode can be set to optimize for static images or video. The video-optimized model is designed for real-time frame-by-frame stylization with temporal consistency, and achieves up to 120 fps on A13 Bionic using the Apple Neural Engine.

**Style Strength Parameter**
Controls the balance between content and style in the stylized output. Low strength preserves content detail with subtle style influence; high strength applies heavy stylization at the cost of content clarity.

**Style Density Parameter**
Controls the granularity of style features the model learns. Low density (coarse) learns high-level shapes and objects from large grid regions; high density (fine) captures fine-grained textures and color patterns from small grid regions.

**Interactive Training Workflow**
Every 5 training iterations, the Create ML app renders a new stylized checkpoint preview of the validation image, allowing real-time visual monitoring. Snapshots can be taken at any point to capture a `.mlmodel`. Style and content loss graphs show convergence. "Train More" extends training from the last checkpoint.

**Integration with ARKit, Core ML, and Metal**
The exported model (`CVPixelBuffer` in, `CVPixelBuffer` out) is used in real-time video pipelines. The ARKit demo pipeline: `ARFrame.capturedImage` → rescale `CVPixelBuffer` → Core ML model → Metal rendering. ARKit person segmentation separates foreground/background, enabling different style models per layer, blended with Metal.

## APIs & Frameworks

### Create ML **[NEW]**
- Style Transfer template **[NEW]** — new task in the Create ML app
- Training inputs: style image (`MLDataTable` or UI drag-drop), content images directory, optional validation image
- Model parameters: style strength (slider), style density (slider), iterations, optimization target (`image` or `video`) **[NEW]**
- Training checkpoint previews — live validation image stylization every 5 iterations **[NEW]**
- Snapshot capture — exports `.mlmodel` snapshot at any training point
- Style loss and content loss graphs — convergence visualization
- "Train More" capability — extend training from last checkpoint **[NEW]**
- Output: Core ML `.mlmodel` under 1 MB; input/output: `CVPixelBuffer`

### Core ML
- Generated model class — `prediction(input:)` with `CVPixelBuffer` input and output
- Model metadata: input/output layer names, OS availability, model size

### ARKit
- `ARFrame.capturedImage` — `CVPixelBuffer` of the live camera feed
- ARKit person segmentation — `ARMatteGenerator` for person/background separation
- `ARAnchor` — placing virtual objects in the AR scene

### Vision / Metal
- `CVPixelBuffer` rescaling — resize to model's expected input dimensions
- Metal rendering — compositing stylized layers (background + stylized foreground person)

## Code Highlights

No code samples were provided in this session. The workflow is entirely within the Create ML app UI. Integration follows a standard Core ML + CVPixelBuffer pipeline:

```swift
// Typical integration pattern for real-time style transfer
import CoreML
import ARKit

let model = try MyStyleTransferModel(configuration: MLModelConfiguration())

func session(_ session: ARSession, didUpdate frame: ARFrame) {
    let pixelBuffer = frame.capturedImage
    // Resize pixelBuffer to model input size, then:
    let input = MyStyleTransferModelInput(image: resizedPixelBuffer)
    let output = try? model.prediction(input: input)
    // output.stylizedImage is a CVPixelBuffer ready to render with Metal
}
```

## Takeaways
- Style Transfer in Create ML trains image and video style models in minutes with no code; the training app provides live visual feedback every 5 iterations via checkpoint previews of a validation image.
- Style strength controls how aggressively content is overwritten by style; style density controls whether coarse (large shapes) or fine (textures/colors) features are learned — combining both parameters produces a wide creative range.
- Video-optimized models run at up to 120 fps on A13 Bionic via the Apple Neural Engine, making real-time per-frame stylization practical; models are under 1 MB and bundle easily with apps.
- Style Transfer composes naturally with ARKit (person segmentation), Core ML, and Metal: different models can stylize foreground and background separately, then blend the results in a Metal render pass.

---
_Source: WWDC20 Session 10642 page (abstract, chapter summaries, code samples, and resource links)._
