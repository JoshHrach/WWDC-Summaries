# Get to Know Metal Function Pointers
**WWDC20 · Session 10013** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10013/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session introduces function pointers in Metal, a new capability in Metal Shading Language (MSL) and the Metal API that makes GPU shaders extensible and dynamically callable. The `[[visible]]` function attribute marks a shader function so it can be wrapped in an `MTLFunction` object, added to a pipeline, and invoked via a function pointer from another shader — enabling plug-in style architectures in ray tracing, dynamic material systems, and more.

The session covers three compilation models (single compilation, separate/binary precompilation, and incremental compilation), how to group functions into `MTLVisibleFunctionTable` objects for GPU access, and key performance considerations including function groups for optimizer hints, recursive function support via `maxCallStackDepth`, and SIMD-group divergence mitigation through threadgroup-level reordering.

## Key Topics

**Visible Functions**
- New `[[visible]]` attribute marks a shader function as externally callable **[NEW]**
- Visible functions can live in separate Metal files or libraries
- Wrapped CPU-side into `MTLFunction` objects via `library.makeFunction(descriptor:)` or `library.makeFunction(name:)`
- Added to pipeline via `MTLLinkedFunctions` and `MTLComputePipelineDescriptor.linkedFunctions`

**Compilation Models**
- **Single compilation** — functions array on `MTLLinkedFunctions`: visible functions are copied into the pipeline and specialized (link-time optimization); best runtime performance, larger pipeline object, longer creation time
- **Separate (binary) compilation** — `MTLFunctionOptions.compileToBinary` precompiles function to a standalone GPU binary; placed in `binaryFunctions` array; smaller pipeline, faster creation, slight runtime call overhead; binary can be shared across pipelines
- **Incremental compilation** — `MTLComputePipelineDescriptor.supportAddingBinaryFunctions = true` allows adding new binary functions to an existing pipeline via `makeComputePipelineStateWithAdditionalBinaryFunctions(functions:)` **[NEW]**; enables dynamic shader streaming (e.g., game asset loading)

**Visible Function Tables**
- `MTLVisibleFunctionTable` — GPU-side array of function pointers **[NEW]**
- Bound to compute encoders or argument buffers at buffer binding points
- In MSL: `visible_function_table<FunctionType>` parameter type; index into it with `[]` to get a function pointer or call directly
- CPU-side: allocated from pipeline state, populated with `functionHandle(function:)` handles; handles remain valid for incrementally compiled pipeline descendants

**Function Groups**
- `[[function_groups("name")]]` attribute on the call-site expression in MSL shader
- CPU-side: `MTLLinkedFunctions.groups` dictionary maps group name to array of possible functions
- Allows the compiler to optimize call sites based on the known set of possible callees, similar to devirtualization

**Recursive Functions**
- Recursive chains of visible function calls now supported **[NEW]**
- `MTLComputePipelineDescriptor.maxCallStackDepth` — set to expected maximum call depth (default: 1); allocates stack accordingly

**Divergence Mitigation**
- In a SIMD group, threads calling different functions serialize execution (worst case: N functions × N threads)
- Mitigation: write function arguments and function index to threadgroup memory, sort by function index, invoke in sorted order, read results from threadgroup memory
- Same pattern works with device memory for cross-threadgroup cases

## APIs & Frameworks

### Metal API (New / Updated)
- `[[visible]]` MSL attribute **[NEW]** — marks function as externally callable
- `MTLFunctionDescriptor` **[NEW]** — descriptor for creating `MTLFunction` objects with options
  - `.name` — function name in library
  - `.options` — `MTLFunctionOptions.compileToBinary` for binary precompilation **[NEW]**
- `library.makeFunction(descriptor: MTLFunctionDescriptor)` **[NEW]** — creates function from descriptor
- `MTLLinkedFunctions` **[NEW]** — container for functions to link into a pipeline
  - `.functions` — array of `MTLFunction` objects to specialize into the pipeline
  - `.binaryFunctions` — array of precompiled binary `MTLFunction` objects
  - `.groups` — `[String: [MTLFunction]]` dictionary for function group hints
- `MTLComputePipelineDescriptor.linkedFunctions` — assign `MTLLinkedFunctions` before pipeline creation
- `MTLComputePipelineDescriptor.supportAddingBinaryFunctions` **[NEW]** — enable incremental compilation on initial pipeline
- `MTLComputePipelineDescriptor.maxCallStackDepth` **[NEW]** — maximum recursive call depth (default: 1)
- `MTLComputePipelineState.functionHandle(function:)` **[NEW]** — returns opaque handle for a visible function
- `MTLComputePipelineState.makeComputePipelineStateWithAdditionalBinaryFunctions(functions:)` **[NEW]** — incremental pipeline creation
- `MTLVisibleFunctionTableDescriptor` **[NEW]** — descriptor with `.functionCount`
- `MTLComputePipelineState.makeVisibleFunctionTable(descriptor:)` **[NEW]** — allocates visible function table
- `MTLVisibleFunctionTable.setFunction(_:index:)` **[NEW]** — populate table entry with function handle
- `MTLComputeCommandEncoder.setVisibleFunctionTable(_:bufferIndex:)` **[NEW]** — bind table to encoder
- `MTLArgumentEncoder.setVisibleFunctionTable(_:index:)` **[NEW]** — bind table in argument buffer

### Metal Shading Language
- `visible_function_table<FunctionType>` **[NEW]** — GPU-side visible function table type
- `[[function_groups("name")]]` **[NEW]** — call-site attribute for optimizer group hint
- Function pointer syntax: `FunctionType *fp = table[index]; fp(args...)` or `table[index](args...)`

## Code Highlights

Single compilation pipeline:
```swift
let linkedFunctions = MTLLinkedFunctions()
linkedFunctions.functions = [area, spot, sphere, hair, glass, skin]
computeDescriptor.linkedFunctions = linkedFunctions
let pipeline = try device.makeComputePipelineState(descriptor: computeDescriptor,
                                                   options: [], reflection: nil)
```

Binary precompilation for separate compilation:
```swift
let functionDescriptor = MTLFunctionDescriptor()
functionDescriptor.name = "Area"
functionDescriptor.options = MTLFunctionOptions.compileToBinary
let areaBinaryFunction = try library.makeFunction(descriptor: functionDescriptor)

let linkedFunctions = MTLLinkedFunctions()
linkedFunctions.binaryFunctions = [areaBinaryFunction]
computeDescriptor.linkedFunctions = linkedFunctions
```

Incremental compilation:
```swift
computeDescriptor.supportAddingBinaryFunctions = true
// later, when new shader asset loads:
let functionDescriptor = MTLFunctionDescriptor()
functionDescriptor.name = "Wood"
functionDescriptor.options = MTLFunctionOptions.compileToBinary
let wood = try library.makeFunction(descriptor: functionDescriptor)
let newPipeline = try pipeline.makeComputePipelineStateWithAdditionalBinaryFunctions(functions: [wood])
```

Visible function table (CPU):
```swift
let vftDescriptor = MTLVisibleFunctionTableDescriptor()
vftDescriptor.functionCount = 3
let lightingFunctionTable = pipeline.makeVisibleFunctionTable(descriptor: vftDescriptor)!
let handle = pipeline.functionHandle(function: spot)!
lightingFunctionTable.setFunction(handle, index: 0)
computeCommandEncoder.setVisibleFunctionTable(lightingFunctionTable, bufferIndex: 1)
```

Visible function table (GPU / MSL):
```metal
using LightingFunction = Lighting(Light, TriangleIntersectionData);
// in kernel parameter list:
visible_function_table<LightingFunction> lightingFunctions [[buffer(1)]]
// usage:
LightingFunction *fn = lightingFunctions[light.index];
Lighting lighting = fn(light, triangleIntersection);
```

Function groups:
```swift
linkedFunctions.groups = ["lighting": [area, spot, sphere],
                          "material": [hair, glass, skin]]
```

## Takeaways
- The `[[visible]]` attribute and `MTLLinkedFunctions` enable a plug-in shader architecture: material and lighting functions can be authored independently and wired together at runtime without recompiling the entire pipeline.
- Choose the compilation model based on constraints: single compilation for best runtime performance; binary precompilation for fast pipeline creation and cross-pipeline sharing; incremental compilation for dynamic shader streaming.
- `MTLVisibleFunctionTable` is the mechanism for passing arrays of function pointers to the GPU — use `functionHandle(function:)` to populate them and re-use handles across incremental pipeline descendants.
- Mitigate SIMD divergence from function pointers by sorting function calls in threadgroup memory so adjacent threads in a SIMD group are more likely to call the same function.

---
_Source: WWDC20 Session 10013 page (abstract, transcript, code samples, and resource links)._
