# Explore the machine learning development experience
**WWDC22 · Session 10017** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10017/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
This session follows an end-to-end ML development journey: finding an open-source photo colorization model (PyTorch), converting it to Core ML format with coremltools, verifying the conversion, evaluating on-device performance with the new Xcode 14 Core ML Performance Report, integrating into a Swift app using Core Image and Core ML, and optimizing runtime behavior with the new Core ML Instruments template.

The session illustrates the complete loop: model discovery → conversion → verification → performance assessment → Swift integration → Instruments profiling → model re-architecture for speed → re-training with PyTorch on Metal.

## Key Topics

**Model discovery and conversion** — Find open-source models from scientific publications or model hubs. Convert PyTorch models to Core ML `.mlpackage` format using `coremltools.convert` with `ct.TensorType` inputs. Verify the converted model in Python before device integration.

**Core ML Performance Report (Xcode 14 new)** — Drag and drop a `.mlpackage` into Xcode to generate a time-based performance analysis showing estimated prediction time broken down by compute unit (Neural Engine, CPU, GPU). Helps decide whether a model is fast enough before writing any Swift code. The original colorization model ran ~90ms; a re-architected version ran ~16ms entirely on the Neural Engine.

**Core Image LAB color space filters (new)** — `CIFilter.convertRGBtoLab()` and `CIFilter.convertLabToRGB()` (new in iOS 16) enable round-trip conversion between RGB and the CIE LAB color space directly in Core Image, eliminating the need for custom Metal kernels for color space conversion.

**MLShapedArray** — Core ML model outputs can be consumed as `MLShapedArray<Float>` (introduced in iOS 16), enabling direct extraction of model output tensor data without going through `MLMultiArray`.

**Instruments Core ML template (new)** — New Core ML template in Instruments visualizes prediction requests, Neural Engine execution, and identifies cascading prediction queues. Used to detect an accumulation bug (predictions stacking up because a new request was dispatched before the previous one finished) and confirm the fix.

**PyTorch on Metal** — Re-training models on Apple Silicon Mac using GPU acceleration via the new Metal back-end for PyTorch, enabling faster iteration on model architecture changes.

**Vision integration** — `VNDetectRectangleRequest` used to isolate a photo document in a live camera feed before passing to the Core ML colorization model, demonstrating Vision + Core ML pipeline.

## APIs & Frameworks

### Core ML
- `MLModel` — loaded Core ML model
- `MLShapedArray<Scalar>` **[NEW]** — typed array for accessing model output tensors directly
- `MLShapedArray.scalars` — flattened `[Scalar]` values from the shaped array
- Core ML Performance Report **[NEW in Xcode 14]** — time-based analysis per compute unit in Xcode
- Core ML Instruments template **[NEW]** — timeline view of Core ML predictions in Instruments

### Core Image (new filters)
- `CIFilter.convertRGBtoLab()` **[NEW]** — converts RGB `CIImage` to CIE LAB color space
- `CIFilter.convertLabToRGB()` (referenced as `convertLabToRGBFilter()`) **[NEW]** — converts LAB `CIImage` back to RGB
- `CIFilter.colorMatrix()` — used to isolate individual color channels
- `CIColorKernel` — custom Metal-backed kernel for combining L, a*, b* channels into a single `CIImage`
- `CIKernel.kernels(withMetalString:)` — compile a Metal-string kernel at runtime
- `CIImage(bitmapData:bytesPerRow:size:format:colorSpace:)` — create `CIImage` from raw float data
- `CIFormat.Lh` — 16-bit float single-channel format

### coremltools (Python)
- `ct.convert(traced_model, inputs: [ct.TensorType(name:shape:)])` — convert traced PyTorch model to Core ML
- `ct.TensorType` — define Core ML input tensor type and shape
- `coreml_model.predict({input: array})` — run prediction with the converted model in Python for verification
- `coreml_model.save("Model.mlpackage")` — save as `.mlpackage` bundle

### Vision
- `VNDetectRectangleRequest` — detect rectangular documents in a camera frame for cropping before ML inference

### PyTorch on Metal
- Metal back-end for PyTorch on Apple Silicon Macs — GPU-accelerated model training/re-training

## Code Highlights

Converting a PyTorch model to Core ML:
```python
import coremltools as ct
import torch
import Colorizer

torch_model = Colorizer().eval()
example_input = torch.rand([1, 1, 256, 256])
traced_model = torch.jit.trace(torch_model, example_input)
coreml_model = ct.convert(traced_model,
                          inputs=[ct.TensorType(name="input", shape=example_input.shape)])
coreml_model.save("Colorizer.mlpackage")
```

Extracting lightness channel using new `CIFilter.convertRGBtoLab()`:
```swift
let rgbToLabFilter = CIFilter.convertRGBtoLab()
rgbToLabFilter.inputImage = inputImage
rgbToLabFilter.normalize = true
let labImage = rgbToLabFilter.outputImage!
```

Accessing model output as `MLShapedArray`:
```swift
let outA: [Float] = output.output_aShapedArray.scalars
let outB: [Float] = output.output_bShapedArray.scalars
```

Custom CIKernel combining LAB channels:
```metal
[[stichable]] float4 labCombine(coreimage::sample_t imL,
                                coreimage::sample_t imA,
                                coreimage::sample_t imB) {
    return float4(imL.r, imA.r, imB.r, imL.a);
}
```

## Takeaways
- Use the new Xcode 14 Core ML Performance Report before writing any Swift integration code — it identifies whether a model will run on the Neural Engine and gives a realistic on-device time estimate.
- Use the new Core ML Instruments template to detect runtime issues like cascading prediction queues; ensure each prediction completes before dispatching the next one for real-time pipelines.
- New `CIFilter.convertRGBtoLab()` and `CIFilter.convertLabToRGBFilter()` eliminate custom color-space conversion code for LAB workflows.
- `MLShapedArray` provides typed, ergonomic access to multi-dimensional model output tensors without the lower-level `MLMultiArray` API.

---
_Source: WWDC22 Session 10017 page (abstract, chapter summaries, code samples, and resource links)._
