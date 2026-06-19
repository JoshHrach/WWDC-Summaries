# Build Metal-based Core Image Kernels with Xcode
**WWDC20 · Session 10021** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10021/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
Core Image kernel code can be written in the Metal Shading Language (MSL) instead of the older Core Image Kernel Language (CIKL). This session covers the full workflow for adding Metal-based CIKernels to an Xcode project: configuring custom build rules, authoring `.ci.metal` source files, and loading and applying the compiled kernels at runtime. Writing kernels in Metal eliminates runtime compilation overhead, enables hardware-accelerated features (gather-reads, group-writes, half-float math), and provides syntax highlighting and compile-time error checking in Xcode.

The demo kernel implements an "HDR Zebra" effect — an animated diagonal stripe overlay that highlights extended-range HDR pixels (values above 1.0) in a video frame — the same effect shown in the companion session "Edit and Playback HDR Video with AVFoundation." The workflow is the same for any `CIColorKernel`, `CIWarpKernel`, or `CIKernel` type.

All the complexity of the old CIKL-based workflow (runtime string-compiled kernels, no type checking, slow startup) is replaced by a compile-once-at-build-time workflow using two custom Xcode build rules and a pair of compiler flags.

## Key Topics

**Custom Build Rules for `.ci.metal` Files**
Core Image Metal kernels require special compiler and linker flags that are not used for regular Metal compute/graphics shaders. Two custom Xcode build rules must be added to project targets:
1. A rule for `*.ci.metal` files that runs `xcrun metal -fcikernel ...` to produce `.ci.air` output.
2. A rule for `*.ci.air` files that runs `xcrun metallib -cikernel ...` to produce `.ci.metallib` in the app's resources bundle.

These rules ensure the `-fcikernel` flag is applied at both compilation and linking stages, which is required for Core Image to recognize the kernel functions.

**Authoring `.ci.metal` Source**
The Metal source file must include `<CoreImage/CoreImage.h>` (which pulls in `CIKernelMetalLib.h`). Kernel functions must be declared `extern "C"`. The return type and argument types correspond to the CIKernel subclass:
- `CIColorKernel` — returns `float4`, first argument is `coreimage::sample_t`, last is `coreimage::destination`
- `CIWarpKernel` — returns `float2` (destination coordinate), takes `coreimage::destination`
- `CIKernel` — general-purpose, can access input pixels with sampler types

The `coreimage::destination` type provides `coord()`, returning the pixel's output position, enabling positional effects like the zebra stripe.

**Loading Kernels with a Static Property**
In Swift, `CIKernel` objects should be loaded from the compiled `.ci.metallib` resource only once, using a `static` stored property on the `CIFilter` subclass. This defers the load to first use and avoids repeated `Data(contentsOf:)` and `CIColorKernel(functionName:fromMetalLibraryData:)` calls.

**Applying Kernels to Create CIImages**
The filter's `outputImage` computed property calls `kernel.apply(extent:arguments:)` with the input extent and any additional parameters (floats, images, etc.) to produce a new `CIImage`. Core Image handles tiling and graph concatenation automatically.

## APIs & Frameworks

### Core Image
- `CIFilter` — base class for custom image filters; subclass for each kernel
- `CIKernel` — base class for Core Image kernel objects
- `CIColorKernel` — pixel-by-pixel color transform; no access to neighboring pixels
  - `CIColorKernel(functionName:fromMetalLibraryData:)` **[NEW]** — loads a kernel from a compiled `.ci.metallib` Data object
  - `CIColorKernel.apply(extent:arguments:)` — applies the kernel and returns a `CIImage`
- `CIWarpKernel` — remaps pixel positions; used for distortion effects
  - `CIWarpKernel(functionName:fromMetalLibraryData:)` **[NEW]** — loads a warp kernel from Metal metallib data
- `CIKernel(functionName:fromMetalLibraryData:)` **[NEW]** — general-purpose kernel loader from Metal metallib data
- `CIKernel.apply(extent:roiCallback:arguments:)` — general kernel application with ROI callback
- `CIImage` — immutable image object; output of kernel application; compositable
- `CIImage.extent` — bounding rectangle for the image

### Metal Shading Language for Core Image
- `#include <CoreImage/CoreImage.h>` — imports Core Image kernel types
- `coreimage::sample_t` — premultiplied linear float4 RGBA pixel sample from an input image
- `coreimage::destination` — destination output descriptor
  - `dest.coord()` — returns `float2` pixel coordinates in the output image
- `extern "C"` — required declaration linkage for CIKernel functions
- Gather-reads — access neighboring pixels efficiently (advanced feature)
- Group-writes — write to a 2D tile of output pixels at once (advanced feature)
- Half-float math (`half`, `half4`) — for performance-sensitive kernels

### Xcode Build System
- Custom Build Rule for `*.ci.metal`: script: `xcrun metal -fcikernel "$INPUT_FILE_PATH" -o "$DERIVED_FILE_DIR/$INPUT_FILE_BASE.ci.air"`; output: `$(DERIVED_FILE_DIR)/$(INPUT_FILE_BASE).ci.air`
- Custom Build Rule for `*.ci.air`: script: `xcrun metallib -cikernel "$INPUT_FILE_PATH" -o "$TARGET_BUILD_DIR/$UNLOCALIZED_RESOURCES_FOLDER_PATH/$INPUT_FILE_BASE.ci.metallib"`; output: `$(UNLOCALIZED_RESOURCES_FOLDER_PATH)/$(INPUT_FILE_BASE).ci.metallib`
- `-fcikernel` — compiler flag required for Core Image Metal kernels (both compile and link stages)
- Syntax highlighting and compile-time error checking — provided automatically in `.ci.metal` files by Xcode

### Foundation
- `Bundle.main.url(forResource:withExtension:)` — locates `*.ci.metallib` in the app bundle
- `Data(contentsOf:)` — loads metallib binary for kernel initialization

## Code Highlights

HDR Zebra kernel in Metal (`MyKernels.ci.metal`):
```metal
#include <CoreImage/CoreImage.h>
using namespace metal;

extern "C" float4 HDRZebra(coreimage::sample_t s, float time, coreimage::destination dest) {
    float diagLine = dest.coord().x + dest.coord().y;
    float zebra = fract(diagLine / 20.0 + time * 2.0);
    if ((zebra > 0.5) && (s.r > 1 || s.g > 1 || s.b > 1))
        return float4(2.0, 0.0, 0.0, 1.0);
    return s;
}
```

Loading and applying the kernel in Swift:
```swift
class HDRZebraFilter: CIFilter {
    var inputImage: CIImage?
    var inputTime: Float = 0.0

    // Load once, lazily, at first use
    static var kernel: CIColorKernel = {
        let url = Bundle.main.url(forResource: "MyKernels", withExtension: "ci.metallib")!
        let data = try! Data(contentsOf: url)
        return try! CIColorKernel(functionName: "HDRZebra", fromMetalLibraryData: data)
    }()

    override var outputImage: CIImage? {
        guard let input = inputImage else { return nil }
        return HDRZebraFilter.kernel.apply(extent: input.extent,
                                           arguments: [input, inputTime])
    }
}
```

## Takeaways
- Add two custom build rules to any target that uses Core Image Metal kernels: one for `.ci.metal` (compile with `-fcikernel`) and one for `.ci.air` (link with `-cikernel`); this shifts compilation from app launch to build time.
- Kernel functions must be declared `extern "C"` and include `<CoreImage/CoreImage.h>` to access `coreimage::sample_t` and `coreimage::destination`; function signatures must match the chosen `CIKernel` subclass type.
- Load `CIKernel` objects using a `static` property on the `CIFilter` subclass so the `.ci.metallib` resource is read and parsed only once per session, avoiding repeated I/O in the filter's `outputImage` getter.
- Metal-based CIKernels unlock gather-reads, group-writes, and half-float arithmetic that are unavailable in the string-based CIKL; they also provide Xcode syntax highlighting and compile-time error reporting.

---
_Source: WWDC20 Session 10021 page (transcript, code samples, and resource links)._
