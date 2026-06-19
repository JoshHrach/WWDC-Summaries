# Discover Metal debugging, profiling, and asset creation tools
**WWDC21 · Session 10157** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10157/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12 — Xcode 13

## Overview
Xcode 13 brings a wave of Metal Debugger improvements across four areas: expanded Metal feature support (ray tracing, function pointers, dynamic libraries), new profiling workflows (GPU Timeline and consistent GPU performance state), debugging workflow refinements (shader validation for indirect command buffers, precise capture controls, separate debug information files, selective shader debugging), and a brand-new **TextureConverter** tool that replaces the old TextureTool with a fully configurable, gamma-aware texture processing pipeline.

## Key Topics

### Acceleration Structure Viewer **[NEW in Xcode 13]**
- New tool in Metal Debugger for inspecting ray tracing acceleration structures captured in a GPU trace.
- Displays scene geometry on the right and a scene outline on the left; click instances to select them and view their transformation matrix and properties.
- Multiple highlighting modes: **bounding volume traversals** (heat-map of BVH traversal cost) and others; hover tooltip shows traversal and intersection counts.
- Configurable traversal settings matching those on an `MTLIntersector` object in shaders.
- Shader validation now covers ray tracing kernels, function pointers, dynamic libraries, and indirect command buffers.

### GPU Timeline **[NEW in Xcode 13]**
- New unified profiling tool in the Metal Debugger Performance panel that combines Metal system trace timing with GPU counter data.
- Shows parallel Vertex, Fragment, and Compute encoder tracks, reflecting Apple GPU's tile-based deferred rendering (multiple passes run simultaneously).
- Each encoder track expands to show aggregated shader timeline, then individual shader waterfall.
- Counters sub-panel: Occupancy, Bandwidth, Limiter counters correlated with encoder selection.
- Selecting an encoder highlights its active time ranges across all tracks and shows attachments in the sidebar.

### Consistent GPU Performance State **[NEW in Xcode 13]**
- GPU performance state is managed by the OS based on thermals, utilization, and settings; fluctuations distort profiling results.
- Three new ways to see and control it:
  1. **Instruments Metal system trace**: new GPU performance state track; "Recording Options" lets you induce a specific state before capture.
  2. **Metal Debugger**: "Stopwatch" button re-profiles the captured trace at a selected performance state; summary page shows the selected state and updated metrics.
  3. **Device Conditions (Xcode 13)**: new "GPU Performance State" device condition in Window → Devices and Simulators forces a specified state for the duration of the connection — works for any testing scenario, not just profiling.

### Precise Capture Controls **[NEW in Xcode 13]**
- New Metal-logo Capture button in the debug bar with a context menu offering:
  - Number of frames to capture (1–5).
  - Capture by command buffers sharing a parent device/command queue, or by MTLLayer presentation.
  - Custom scopes via `MTLCaptureScope` API.
- Pipeline State Viewer: selecting a draw call now shows the active pipeline state in Bound Resources; clicking opens the viewer for functions, properties, memory usage.
- Memory Viewer: now shows pipeline state memory allocation sizes.

### Separate Debug Information Files **[NEW in Xcode 13]**
- Previously: shaders needed sources embedded in Metallib for debugging (not allowed in App Store distribution) → required two build variants.
- New: compile a Metallib with `--record-sources flat` flag to generate a companion `.metallibsym` file containing sources and debug info separately from the library itself.
- In Metal Debugger: when debugging a shader without embedded sources, a dialog prompts to import `.metallibsym` files; after import, libraries and sources are matched automatically.
- Allows shipping a release Metallib without embedded sources while retaining the ability to debug against the same binary.

### Selective Shader Debugging **[NEW in Xcode 13]**
- Large shaders can take a long time to start the shader debugger.
- Right-click any function in the shader editor → "Debug Functions" → pre-selects the function scope; debugger starts almost instantly for that function only.

### TextureConverter — New Texture Compression Tool **[NEW in Xcode 13]**
Replaces TextureTool with a fully-configurable, gamma-aware pipeline. Available for macOS and Windows (Metal Developer Tools for Windows 2.0).

Pipeline stages (all configurable):
1. **Gamma input** (`--gamma_in`): convert input to working color space (linear float or sRGB).
2. **Physical transforms**: downscale with `max_extent`, resize filter, flip axes.
3. **Mipmap generation**: max count, filter (Kaiser default, box, triangle).
4. **Alpha handling**: alpha-to-coverage, discard/preserve/premultiply.
5. **Gamma output** (`--gamma_out`): convert to target color space.
6. **Channel mapping** (optional):
   - **RGBM encoding**: compresses HDR data into LDR RGBA by storing a scale multiplier in alpha; controlled by `RGBM_Range` (default 6.0).
   - **Normal map encoding**: remaps X→red/blue, Y→alpha/green per format for better quality; complement with `MTLTextureSwizzleChannelsMake` at runtime for format-neutral sampling.
7. **Compression**: `--compression_format` selects BCn (macOS), ASTC (iOS/Apple Silicon, recommended), or PVRTC (legacy A7 and earlier). Compression quality level (trade-off between speed and quality).

Compatibility mode: pass old TextureTool flags to `xcrun TextureConverter`; it translates them and prints the native equivalents to ease migration.

## APIs & Frameworks

**Xcode 13 Metal Debugger**
- Acceleration Structure Viewer — inspect ray tracing BVH geometry and traversal cost **[NEW]**
- GPU Timeline in Performance panel — parallel encoder tracks + counters **[NEW]**
- GPU Performance State control (Stopwatch button in Metal Debugger) **[NEW]**
- GPU Performance State device condition (Devices and Simulators) **[NEW]**
- Instruments Metal system trace GPU performance state track **[NEW]**
- Separate debug information: `--record-sources flat` compiler flag → `.metallibsym` files **[NEW]**
- Selective shader debugging via right-click "Debug Functions" **[NEW]**
- Shader validation for ICBs, dynamic libraries, function pointers **[extended]**
- Precise capture controls via Metal-logo button (frame count, scope) **[NEW]**
- Pipeline State Viewer accessible from bound resources in draw call **[NEW/improved]**
- Memory Viewer: pipeline state memory sizes **[NEW]**

**Metal Shading Language / Texture Processing**
- `xcrun TextureConverter` — new texture compression CLI tool **[NEW]**
- `MTLTextureDescriptor.swizzle` + `MTLTextureSwizzleChannelsMake` — runtime channel remapping for format-neutral normal sampling **[existing]**

## Code Highlights

RGBM encoding in a Metal shader (HDR → LDR RGBA with alpha scale):
```metal
float4 EncodeRGBM(float3 in) {
    float4 rgbm;
    rgbm.a = max3(in.r, in.g, in.b) / RGBM_Range;
    rgbm.rgb = in / (rgbm.a * RGBM_Range);
    return rgbm;
}

float3 DecodeRGBM(float4 sample) {
    const float RGBM_Range = 6.0f;
    float scale = sample.a * RGBM_Range;
    return sample.rgb * scale;
}
```

Runtime texture swizzle to read ASTC normal map X from red, Y from alpha:
```objc
MTLTextureDescriptor *descriptor = [[MTLTextureDescriptor alloc] init];
descriptor.swizzle = MTLTextureSwizzleChannelsMake(
    MTLTextureSwizzleRed,    // X → red
    MTLTextureSwizzleAlpha,  // Y → alpha
    MTLTextureSwizzleZero,
    MTLTextureSwizzleZero
);
```

Reconstruct normal Z from XY in a shader:
```metal
float3 ReconstructNormal(float2 sample) {
    float3 normal;
    normal.xy = sample.xy * 2.0f - 1.0f;
    normal.z = sqrt(saturate(1.0f - dot(normal.xy, normal.xy)));
    return normal;
}
```

Generate a `.metallibsym` separate debug information file at build time:
```bash
xcrun -sdk iphoneos metal --record-sources flat -o Shaders.metallib Shaders.metal
# Produces: Shaders.metallib + Shaders.metallibsym
```

## Takeaways
- The GPU Timeline gives a single unified view of parallel encoder execution on Apple GPUs—use it to see how vertex, fragment, and compute work actually interleave on the hardware.
- Always lock down GPU performance state before benchmarking; use Device Conditions to force a state without touching profiling tools.
- Separate `.metallibsym` files allow debugging any release Metallib without embedding sources; adopt `--record-sources flat` in debug build scripts immediately.
- TextureConverter replaces TextureTool: migrate with `xcrun TextureConverter --compat` to translate old command lines automatically.
- Prefer ASTC over PVRTC for all new content on iOS 15 and Apple Silicon; use BCn on Intel macOS targets.

---
_Source: WWDC21 Session 10157 page (abstract, full transcript, code samples, and resource links)._
