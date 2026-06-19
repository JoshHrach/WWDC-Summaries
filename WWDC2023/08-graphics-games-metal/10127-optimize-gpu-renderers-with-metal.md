# Optimize GPU renderers with Metal
**WWDC23 · Session 10127** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10127/)

_Platforms:_ iOS 16.4+, macOS 13.3+, tvOS, visionOS 1

## Overview
Modern digital content creation apps and game engines must balance two competing goals: responsive material authoring (so artists see changes instantly without shader recompilation hitches) and maximum rendering performance. This session walks through the Metal features and best practices that let you achieve both simultaneously, using Blender 3D's Cycles path tracer as a real-world example.

The session covers four main areas: maximizing shader performance through function specialization (function constants), keeping applications responsive via asynchronous compilation, reducing compile time at runtime with Metal dynamic libraries, and tuning compute shaders for optimal GPU occupancy using new Metal compiler options.

Combining all four techniques produced dramatic results in Blender 3D — material rendering improved from 58 ms to 12.5 ms per frame on the Wanderer and Tree Creature test assets, and specialized material variants became available much sooner during multi-material authoring workflows.

## Key Topics

### Maximize Shader Performance with Function Specialization
Uber shaders handle every possible material combination via runtime branches, which is responsive but slow. Metal function constants let you replace dynamic branches with compile-time constants, eliminating dead code and memory loads. The compiler folds false branches entirely, leaving only the active codepath.

### Asynchronous Compilation
Blocking on shader compilation causes hitches. Metal's async pipeline-creation APIs (completion handler variants) return immediately so the UI stays responsive. The app renders with the uber shader while the optimized variant compiles in the background, then switches automatically on completion.

### Maximize Concurrent Compilation
The new `shouldMaximizeConcurrentCompilation` device property (macOS 13.3) tells the Metal compiler to saturate available CPU cores with parallel compile jobs — critical for multi-material authoring workflows where many shaders are invalidated simultaneously.

### Fast Runtime Compilation via Dynamic Libraries
Splitting utility functions (math, lighting, shadows) into Metal dynamic libraries (`MTLDynamicLibrary`) allows them to be pre-compiled offline and reused across pipeline states. Runtime compilation then only needs to compile the thin material-specific shader, dramatically reducing latency.

### Tune Compute Shaders with Occupancy Hints
`maxTotalThreadsPerThreadgroup` on `MTLComputePipelineDescriptor` controls the compiler's occupancy target. A new matching property on `MTLCompileOptions` (iOS 16.4 / macOS 13.3) lets dynamic libraries be compiled to the same occupancy target as the pipeline state that uses them, finding the performance sweet spot for complex kernels like path tracers.

## APIs & Frameworks

- **Metal** — primary framework throughout
- `[[function_constant(N)]]` — Metal Shading Language attribute for declaring compile-time constants **[highlight feature]**
- `MTLFunctionConstantValues` — host-side container for function constant values
  - `-setConstantValue:type:withName:` — sets a constant by name
- `MTLLibrary` — compiled shader library
  - `-newFunctionWithName:constantValues:error:` — creates a specialized function variant **[key API]**
- `MTLRenderPipelineDescriptor`
  - `fragmentFunction` / `vertexFunction`
  - `fragmentPreloadedLibraries` / `vertexPreloadedLibraries` — for dynamic library linking **[NEW]**
- `MTLDevice`
  - `-newLibraryWithSource:options:completionHandler:` — async library creation
  - `-newRenderPipelineStateWithDescriptor:completionHandler:` — async PSO creation
  - `shouldMaximizeConcurrentCompilation` — maximize parallel shader compilation **[NEW]** (macOS 13.3)
  - `supportsRenderDynamicLibraries` — capability query
  - `supportsDynamicLibraries` — capability query (compute)
  - `-newDynamicLibrary:error:` — create dynamic library from MTLLibrary
  - `-newDynamicLibraryWithURL:error:` — load pre-compiled dynamic library from disk
- `MTLDynamicLibrary` — pre-compiled reusable Metal library
- `MTLCompileOptions`
  - `libraryType` (`MTLLibraryTypeDynamic`) — compile as dynamic library
  - `installName` — install path for the dynamic library
  - `libraries` — array of dynamic libraries to link against
  - `maxTotalThreadsPerThreadgroup` — occupancy hint for dynamic libraries **[NEW]** (iOS 16.4, macOS 13.3)
- `MTLComputePipelineDescriptor`
  - `maxTotalThreadsPerThreadgroup` — target occupancy hint for compute kernels
- Symbol visibility attributes: `__attribute__((visibility("default")))` / `__attribute__((visibility("hidden")))`
- Xcode GPU Debugger — Performance section for ALU instruction counts, spill, and memory wait analysis

## Code Highlights

Declaring function constants in Metal Shading Language to replace runtime branches:
```metal
constant bool IsGlossy       [[function_constant(0)]];
constant bool HasShadows     [[function_constant(1)]];
constant bool HasReflections [[function_constant(2)]];
constant bool IsVolumetric   [[function_constant(3)]];

fragment FragOut frag_material_main(device Material &material [[buffer(0)]]) {
    if(IsGlossy)       { material_glossy(material); }
    if(HasShadows)     { light_shadows(material); }
    if(HasReflections) { trace_reflections(material); }
    if(IsVolumetric)   { output_volume_parameters(material); }
    return output_material();
}
```

Creating a specialized PSO with function constant values on the host:
```objc
MTLFunctionConstantValues* values = [MTLFunctionConstantValues new];
for(const MaterialParameter& p : shader_parameters) {
    [values setConstantValue:p.value_ptr type:p.type withName:p.name];
}
dsc.fragmentFunction = [shader_library newFunctionWithName:@"frag_material_main"
                                             constantValues:values
                                                      error:&error];
```

Setting the occupancy hint on MTLCompileOptions for dynamic libraries:
```objc
MTLCompileOptions* options = [MTLCompileOptions new];
options.libraryType = MTLLibraryTypeDynamic;
options.installName = @"executable_path/dylib_Math.metallib";
if(@available(macOS 13.3, *)) {
    options.maxTotalThreadsPerThreadgroup = 768;
}
```

## Takeaways

- Replace uber-shader runtime branches with Metal function constants to eliminate dead code and dramatically reduce GPU time (58 ms → 12.5 ms in the Blender example).
- Use async pipeline state creation APIs so material edits never stall the UI; render with the uber shader in the interim.
- Pre-compile utility code into Metal dynamic libraries to minimize runtime compilation cost; match `maxTotalThreadsPerThreadgroup` between PSO and dylibs.
- Profile with Xcode GPU Debugger Performance section and experiment with `maxTotalThreadsPerThreadgroup` values per device — the optimal value is workload- and architecture-specific.

---
_Source: WWDC23 Session 10127 page (abstract, chapter summaries, code samples, and resource links)._
