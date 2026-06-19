# Explore Core Image Kernel Improvements
**WWDC21 · Session 10159** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10159/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This session covers the two recommended ways to add custom Metal-based Core Image kernels to an Xcode project in 2021: the established "extern" method using custom Xcode build rules and `.ci.metal` / `.ci.metallib` file conventions, and the new "stitchable" method introduced in iOS 15 and macOS 12 that uses Metal's `[[stitchable]]` function attribute and Metal Dynamic Libraries.

Both methods shift kernel compilation from app launch time to build time, improving cold-start performance. Writing kernels in Metal (rather than the legacy CISL string-based API) also unlocks gather-reads, group-writes, half-float math, Xcode syntax highlighting, and inline compile-time error checking.

## Key Topics
- **Benefits of Metal CIKernels:** Compile-time kernel compilation (vs. runtime), automatic tiling and concatenation by Core Image, access to gather-reads, group-writes, and half-float math; Xcode syntax highlighting and inline error checking.
- **Extern Method (Established):** Add two custom Xcode build rules: one for `.ci.metal` → `.ci.air` (compiler flag: `-fcikernel`) and one for `.ci.air` → `.ci.metallib` (linker flag: `-cikernel`). Kernel functions must be declared `extern "C"`. Load from bundle using the `.ci.metallib` resource extension.
- **Stitchable Method (NEW in iOS 15 / macOS 12):** No custom build rules required. Set `Other Metal Linker Flags` to `-framework CoreImage`. Kernel functions use `[[stitchable]]` attribute instead of `extern "C"`. Load from the standard `default.metallib` bundle resource. Supports linking against other Metal libraries and accepts integer/unsigned integer vector input parameters.
- **Device Compatibility for Stitchable Kernels:** Requires `MTLDevice.supportsDynamicLibraries == true`. Supported on iPhone/iPad with A11+, all Apple silicon Macs, and Intel Macs with AMD Navi and Vega GPUs. Always check before using stitchable kernels.
- **Runtime Compilation (Niche Use):** Stitchable kernels can also be compiled from source at runtime, but this incurs longer initial compile time and should only be used in applications that specifically require runtime kernel flexibility.
- **CIFilter Subclass Pattern:** Instantiate `CIKernel` as a `static` property on the filter class to ensure the metallib resource is loaded only once. Override `outputImage` and call `kernel.apply(extent:arguments:)`.

## APIs & Frameworks

**Core Image**
- `CIKernel` – Base class for custom Core Image kernel objects (existing)
- `CIColorKernel` – Subclass for per-pixel color kernels (existing)
- `CIWarpKernel` – Subclass for geometry-distorting kernels (existing)
- `CIBlendKernel` – Subclass for two-image blend kernels (existing)
- `CIKernel(functionName:fromMetalLibraryData:)` – Loads a Metal-compiled kernel by function name from `Data` (existing)
- `CIColorKernel(functionName:fromMetalLibraryData:)` – Same for color kernels (existing)
- `CIKernel.apply(extent:roiCallback:arguments:)` – Applies the kernel to produce a new `CIImage` (existing)
- `CIKernel.apply(extent:arguments:)` – Simplified apply for color kernels (existing)
- `#include <CoreImage/CoreImage.h>` – MSL header providing `coreimage::sample_t`, `coreimage::destination`, and related types

**Metal / Metal Shading Language**
- `[[stitchable]]` function attribute **[NEW]** – MSL 2.4 attribute; enables Metal Dynamic Library linkage and Core Image recognition
- Metal Shading Language 2.4 **[NEW]** – Required for `[[stitchable]]`
- Metal Dynamic Libraries **[NEW]** – Backend mechanism enabling stitchable kernel linkage against Core Image Metal framework
- `MTLDevice.supportsDynamicLibraries` **[NEW]** – Check before using stitchable kernels
- `-fcikernel` compiler flag – Required for extern method `.ci.metal` → `.ci.air` build rule
- `-cikernel` linker flag – Required for extern method `.ci.air` → `.ci.metallib` build rule
- `-framework CoreImage` – `Other Metal Linker Flags` value required for stitchable method

**MSL Core Image Types**
- `coreimage::sample_t` – Sampled input pixel value (RGBA float4)
- `coreimage::destination` – Output pixel descriptor; `dest.coord()` returns current output pixel coordinates
- `extern "C"` – Required function linkage for extern method
- `[[stitchable]]` – Required attribute for stitchable method

**Xcode Build System**
- Custom Build Rule (`.ci.metal` → `.ci.air`) **[NEW usage pattern]** – Invokes Metal compiler with `-fcikernel`
- Custom Build Rule (`.ci.air` → `.ci.metallib`) **[NEW usage pattern]** – Invokes Metal linker with `-cikernel`
- `Other Metal Linker Flags` build setting – Set to `-framework CoreImage` for stitchable method **[NEW]**

## Code Highlights
Extern CIKernel (MSL source file `MyKernels.ci.metal`):
```metal
#include <CoreImage/CoreImage.h>
using namespace metal;

extern "C" float4 myKernel(coreimage::sample_t s,
                            float param,
                            coreimage::destination dest) {
    float diagLine = dest.coord().x + dest.coord().y;
    float stripe   = fract(diagLine / 20.0 + param * 2.0);
    float4 result  = s;
    if ((stripe > 0.5) && ((s.r > 1) || (s.g > 1) || (s.b > 1)))
        result = float4(2.0, 0.0, 0.0, 1.0);
    return result;
}
```

Loading an extern kernel in Swift:
```swift
class MyFilter: CIFilter {
    var inputImage: CIImage?
    var inputParam: Float = 0.0

    static var kernel: CIColorKernel = {
        let url  = Bundle.main.url(forResource: "MyKernels", withExtension: "ci.metallib")!
        let data = try! Data(contentsOf: url)
        return try! CIColorKernel(functionName: "myKernel", fromMetalLibraryData: data)
    }()

    override var outputImage: CIImage? {
        guard let input = inputImage else { return nil }
        return MyFilter.kernel.apply(extent: input.extent, arguments: [input, inputParam])
    }
}
```

Stitchable CIKernel (MSL, any `.metal` file):
```metal
#include <CoreImage/CoreImage.h>
using namespace metal;

[[stitchable]] float4 myKernel(coreimage::sample_t s,
                               float param,
                               coreimage::destination dest) {
    float diagLine = dest.coord().x + dest.coord().y;
    float stripe   = fract(diagLine / 20.0 + param * 2.0);
    float4 result  = s;
    if ((stripe > 0.5) && ((s.r > 1) || (s.g > 1) || (s.b > 1)))
        result = float4(2.0, 0.0, 0.0, 1.0);
    return result;
}
```

Loading a stitchable kernel in Swift (uses `default.metallib`):
```swift
static var kernel: CIColorKernel = {
    let url  = Bundle.main.url(forResource: "default", withExtension: "metallib")!
    let data = try! Data(contentsOf: url)
    return try! CIColorKernel(functionName: "myKernel", fromMetalLibraryData: data)
}()
```

## Takeaways
- The new `[[stitchable]]` method is simpler to set up (no custom build rules, standard `default.metallib` resource name) and should be preferred for any app targeting iOS 15 / macOS 12 and above on supported GPUs.
- Always check `MTLDevice.supportsDynamicLibraries` before instantiating stitchable kernels; fall back to the extern method for older hardware if needed.
- In both methods, instantiate `CIKernel` as a `static` lazy property to avoid repeated metallib loading.
- Stitchable kernels unlock Metal Dynamic Library linking, meaning kernels can call helper functions defined in other Metal libraries—enabling modular, reusable shader code.

---
_Source: WWDC21 Session 10159 page (abstract, transcript, and code samples)._
