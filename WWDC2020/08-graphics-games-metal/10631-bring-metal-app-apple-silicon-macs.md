# Bring Your Metal App to Apple Silicon Macs
**WWDC20 · Session 10631** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10631/)

_Platforms:_ macOS Big Sur 11 (Apple silicon)

## Overview
Apple silicon Macs use the same Tile-Based Deferred Rendering (TBDR) GPU architecture as iPhone, iPad, and Apple TV — a fundamental shift from the Immediate Mode Rendering (IMR) GPUs in Intel-based Macs. This session explains the architectural differences, the migration path from Intel to Apple silicon, and four common correctness issues that arise when Metal apps move to a TBDR GPU without changes.

Existing x86 binaries run out of the box under Rosetta 2 with automatic Metal consistency workarounds applied, but those workarounds have a performance cost. The goal is to recompile natively for Apple silicon, fix correctness issues, and then take advantage of TBDR-specific features (programmable blending, tile shaders, memoryless framebuffers, local image blocks) that were not previously available on macOS.

## Key Topics
- **TBDR vs. IMR architecture** — IMR (Intel/AMD/Nvidia) processes triangles immediately, requires the entire depth/color buffer to be in cache; TBDR (Apple GPU) divides the screen into tiles, builds a vertex buffer for each tile, then renders all pixels per tile — depth test happens before shading, eliminating hidden surface pixels entirely, all blending happens on-chip without accessing system memory.
- **Migration path** — (1) Run existing x86 binary under Rosetta → automatic Metal consistency workarounds applied, slight perf cost; (2) Recompile natively (Apple silicon target) → workarounds removed, must fix issues manually; (3) Optimize for TBDR (companion session "Optimize Metal Performance for Apple silicon Macs").
- **Unified feature set** — Apple silicon Mac supports both `MTLGPUFamilyMac2` and `MTLGPUFamilyApple5`+ (TBDR-specific features); previously only `MTLGPUFamilyMac2` was available on macOS.
- **Feature detection** — Use `MTLDevice.supportsFamily(_:)` to check for Apple GPU features; use `computePipeline.threadExecutionWidth` to get simdgroup size; use `MTLDevice.isLowPower` to classify the GPU (note: Apple GPUs return `false` — treat them like discrete GPUs despite being power-efficient).
- **Issue 1: Load actions** — On TBDR, `.dontCare` load action means tile memory is NOT initialized from system memory; if the frame buffer was previously rendered, using `.dontCare` produces artifacts. Use `.load` when previous content must be preserved. Use `.dontCare` only for attachments that will be fully overwritten.
- **Issue 2: Store actions** — Use `.store` only if the attachment is consumed in a later pass; using `.store` unnecessarily adds memory traffic; use `.dontCare` for intermediate passes (e.g., depth that is not needed after the render pass).
- **Issue 3: Position invariance** — Apple GPU compilers aggressively optimize vertex shaders; the same position calculation in two different vertex shaders may produce slightly different float values. If a pass compares depth using `equal`, this causes pixels to be discarded incorrectly. Fix: pass `preserveInvariance = true` to `MTLCompileOptions` and mark the position output with `[[invariant]]` in Metal shaders.
- **Issue 4: Threadgroup memory synchronization** — Apple GPUs have a simdgroup size of 32; apps assuming a simdgroup size of 64 (Intel) omit necessary barriers. Query simdgroup size via `threads_per_simdgroup` (shader built-in) or `computePipeline.threadExecutionWidth` (API); use `simdgroup_barrier` when `simd_size == 32` and only one simdgroup per threadgroup, or `threadgroup_barrier` when multiple simdgroups are in play.
- **Issue 5: Sampling current depth/stencil attachment** — Reading the current depth attachment in the same render pass that is writing to it is undefined behavior on any GPU; on TBDR this triggers a correctness race condition as the tile is flushed. Do not use memory/texture barriers to work around this — instead, snapshot the depth texture before the render pass that needs to sample it.
- **Compatibility workarounds (Catalina SDK or earlier)** — When Metal detects an app built against macOS Catalina SDK or earlier, it automatically: remaps `.dontCare` loads to `.load`; forces position invariance on all vertex shaders; snapshots depth textures when they are also bound as samples. These workarounds are NOT applied for apps built with the macOS Big Sur SDK.
- **Metal API Validation** — Augmented in macOS Big Sur to flag incorrect load/store actions and depth texture sampling violations during development.

## APIs & Frameworks

### Metal
- **`MTLDevice.supportsFamily(_:)`** — `func supportsFamily(_ gpuFamily: MTLGPUFamily) -> Bool`; use `MTLGPUFamilyApple5` (or higher) to check for TBDR-specific features on Apple silicon
- **`MTLGPUFamily.apple5`** — Apple GPU family 5 (A13-equivalent); TBDR features including programmable blending, tile shaders, memoryless framebuffers, local image blocks
- **`MTLGPUFamily.mac2`** — Mac GPU family 2; still supported and available on Apple silicon
- **`MTLDevice.isLowPower`** — `var isLowPower: Bool`; returns `false` for Apple GPUs — treat as discrete, not integrated, for performance-level decisions
- **`MTLComputePipelineState.threadExecutionWidth`** — `var threadExecutionWidth: Int`; returns 32 on Apple GPUs; use to determine simdgroup size at runtime
- **`MTLCompileOptions.preserveInvariance`** **[NEW]** — `var preserveInvariance: Bool`; when `true`, compiler preserves position invariance across vertex shaders compiled from the same source; required for multi-pass algorithms with depth compare == equal
- **`MTLRenderPassDescriptor.colorAttachments[n].loadAction`** / **`.storeAction`** — `MTLLoadAction` (`.load`, `.clear`, `.dontCare`) and `MTLStoreAction` (`.store`, `.dontCare`, `.multisampleResolve`, etc.); critical to set correctly on TBDR
- **`MTLRenderPassDescriptor.depthAttachment.loadAction`** / **`.storeAction`** — Same; use `.dontCare` store for depth when not needed in later passes

### Metal Shading Language
- **`[[invariant]]`** attribute on `[[position]]` output — Required in combination with `MTLCompileOptions.preserveInvariance`; marks the position output as invariant across shader compilations
- **`threads_per_simdgroup`** built-in — `uint threads_per_simdgroup [[threads_per_simdgroup]]`; kernel attribute to query simdgroup size at shader execution time; 32 on Apple GPUs
- **`simdgroup_barrier(mem_flags::mem_threadgroup)`** — Use when there is exactly one simdgroup per threadgroup; cheaper than `threadgroup_barrier`
- **`threadgroup_barrier(mem_flags::mem_threadgroup)`** — Required when multiple simdgroups share threadgroup memory

## Code Highlights

API-driven feature detection (never use GPU name strings):
```swift
let supportsAppleGPU = metalDevice.supportsFamily(.apple5)
let simdgroupSize = computePipeline.threadExecutionWidth  // 32 on Apple GPU
let treatAsDiscrete = !metalDevice.isLowPower             // true on Apple GPU
```

Enable position invariance:
```swift
// Swift: compile with preserveInvariance
let options = MTLCompileOptions()
options.preserveInvariance = true
let library = try device.makeLibrary(source: shaderSource, options: options)
```
```metal
// Metal shader: mark position output as invariant
struct VertexOut {
    float4 pos [[position, invariant]];
    float2 uv;
};
```

Correct threadgroup synchronization for any simdgroup size:
```metal
kernel void myKernel(uint tid [[thread_index_in_threadgroup]],
                     uint simd_size [[threads_per_simdgroup]],
                     threadgroup uint* buf [[threadgroup(0)]])
{
    buf[tid] = computeValue(tid);
    if (simd_size == 64u)
        simdgroup_barrier(mem_flags::mem_threadgroup);
    else
        threadgroup_barrier(mem_flags::mem_threadgroup);
    uint result = buf[tid] + buf[(tid + 32) % 64];
}
```

Correct load/store action pattern:
```swift
// Render pass that builds on a previous skybox pass
let rpd = MTLRenderPassDescriptor()
rpd.colorAttachments[0].loadAction  = .load      // preserve skybox
rpd.colorAttachments[0].storeAction = .store      // needed by display
rpd.depthAttachment.loadAction  = .clear
rpd.depthAttachment.storeAction = .dontCare      // depth not consumed later
```

## Takeaways
- Existing Metal apps run under Rosetta on Apple silicon with automatic workarounds, but those workarounds cost performance — native compilation plus manual correctness fixes is the required next step.
- Never detect GPU capabilities by parsing the device name; always use `MTLDevice.supportsFamily(_:)` and query simdgroup size via `threadExecutionWidth` at runtime.
- TBDR load/store actions directly control on-chip tile memory initialization and flushing: use `.dontCare` load only when the attachment will be fully overwritten; use `.dontCare` store only when the attachment is not consumed downstream.
- Set `MTLCompileOptions.preserveInvariance = true` and mark position outputs `[[invariant]]` for any multi-pass algorithm that relies on depth compare == equal across separate render passes.
- Query `threads_per_simdgroup` (or `threadExecutionWidth`) at runtime to choose the correct barrier type; Apple GPUs have a simdgroup size of 32 — code that assumes 64 omits necessary synchronization.

---
_Source: WWDC20 Session 10631 page (abstract, transcript, and code samples)._
