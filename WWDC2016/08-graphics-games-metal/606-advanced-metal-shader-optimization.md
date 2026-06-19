# Advanced Metal Shader Optimization
**WWDC16 · Session 606** · [Watch](https://developer.apple.com/videos/play/wwdc2016/606/)

_Platforms:_ iOS 10, tvOS 10, macOS Sierra 10.12

## Overview
This session is aimed at experienced shader authors who want to extract maximum performance from Metal shaders running on A8 and later GPUs. It covers both high-level structural decisions—address space selection, buffer preloading, fragment function resource writes, and compute kernel design—and low-level optimizations in data types, arithmetic, control flow, and memory access patterns.

The presenters emphasize that low-level shader tuning is the last step, only productive after higher-level draw-call and engine optimizations are in place. Many of the most impactful techniques involve cooperating with the compiler rather than working around it: writing clear, idiomatic Metal so the compiler can apply its full suite of transformations.

A recurring theme is that several costly pitfalls—dynamically indexed stack arrays, integer division by non-constant denominators, and wrong address-space choices—can dwarf all micro-optimizations combined, so recognizing and avoiding them is the first priority.

## Key Topics

### Address Space Selection
- **`device`** address space: read/write, flexible alignment, no size limits; used for per-vertex data and per-instance data with variable counts.
- **`constant`** address space: read-only, size-bounded, tighter alignment; optimized for data with high reuse (matrices, bone data, light structs). Enables constant buffer preloading into dedicated fast registers.

### Buffer Preloading
- **Constant buffer preloading**: compiler places constant-address-space data into special constant registers for faster ALU access. Requires passing arguments by reference (not pointer) so accesses are statically bounded.
- **Vertex buffer preloading**: reuses fixed-function vertex-fetch hardware for buffer loads indexed by `vertex_id` or `instance_id` (with optional divisors). Use the Metal vertex descriptor wherever possible.

### Fragment Function Resource Writes (iOS 10)
- Fragments that perform resource writes cannot be discarded by hidden-surface removal unless `[[early_fragment_tests]]` is enabled.
- Draw resource-writing objects after opaque objects and sort front-to-back to maximize depth/stencil rejection.

### Compute Kernel Design
- Avoid compute thread launch overhead by processing multiple work items per thread, reusing values loaded for one item when processing the next (demonstrated with a separable filter kernel).
- Use `simdgroup_barrier` **[NEW]** instead of `threadgroup_barrier` when thread group fits within a single SIMD group; this is often faster than using a larger thread group with the heavier barrier.

### Data Types
- A8 and later GPUs have 16-bit native register units; `float` costs twice the registers and bandwidth of `half`.
- Prefer `half` for texture reads and interpolants; prefer `short`/`ushort` for thread IDs and small indices.
- Data type conversions between `float` and `half` are typically free on A8+.
- `char` (8-bit) has no native arithmetic; avoid unless necessary.
- C promotion rules apply: multiplying a `half` by a float literal produces a `float` operation—use typed literals (`1.0h`) to keep arithmetic in `half`.

### Arithmetic Optimizations
- Always use Metal built-in functions; negate, `abs()`, and `saturate()` are typically free (GPU instruction modifiers).
- A8+ GPUs are scalar machines; the compiler splits all vector arithmetic—no need to force scalar style, but also no benefit from artificial vectorization.
- Avoid optimizing for Instruction Level Parallelism (ILP); it increases register pressure and is counterproductive on these GPUs.
- Use the ternary operator (`select`) directly; clever bit-trick alternatives confuse the compiler and are slower.
- **Major pitfall**: integer division/modulus by a non-constant, non-literal denominator compiles to hundreds of clock cycles. Division by literal or `[[function_constant]]` values is fast.
- Fast-math is on by default and critical (~50%+ gain); `fma()` built-in can recover some performance when fast-math must be disabled.

### Control Flow
- Uniform control flow (all SIMD lanes take the same path) is fast even if the compiler cannot statically verify it.
- Divergent control flow causes the GPU to run all paths, increasing all bottlenecks.
- Avoid switch fall-throughs; they require code duplication transformations that harm GPU performance.

### Memory Access
- **Major pitfall**: dynamically indexed non-constant stack arrays are extremely slow (demonstrated with a 30% regression from a single 32-byte stack array). The compiler aggressively unrolls loops over stack arrays to eliminate dynamic indexing.
- A8+ GPUs have vector memory units despite scalar arithmetic; one large vector load is faster than multiple scalar loads. As of iOS 10 the compiler auto-vectorizes neighboring loads where possible—help it by placing adjacent fields together in structs or using `float2`/`float4` types directly.
- Device memory address offsets must fit in a signed integer; use `int` or `short`/`ushort` offsets, not `uint`, to allow the dedicated addressing hardware to be used.

### Latency and Occupancy
- GPUs hide latency with many concurrent threads; register usage and threadgroup memory usage limit occupancy and therefore latency hiding.
- Use `MTLComputePipelineState.maxTotalThreadsPerThreadgroup` **[NEW]** to query actual occupancy.
- Serial texture reads (each dependent on the previous) multiply latency; parallel independent reads share a single wait.
- False control-flow dependencies (branching on a value not used by a subsequent texture read) still force a full wait; restructure to hoist independent reads before the branch.
- A8+ typically needs at least two concurrent texture reads to fully hide latency.

## APIs & Frameworks

- **Metal** shading language (MSL)
- `device` address space qualifier
- `constant` address space qualifier **[NEW emphasis]**
- Metal vertex descriptor (`MTLVertexDescriptor`) for vertex buffer preloading
- `[[early_fragment_tests]]` attribute **[NEW in iOS 10]** — enables early depth/stencil rejection for fragment functions with resource writes
- `simdgroup_barrier()` **[NEW in iOS 10]** — lightweight barrier scoped to a single SIMD group
- `threadgroup_barrier()` — existing thread-group-scoped barrier
- `fma(a, b, c)` built-in — fused multiply-add
- `select(a, b, condition)` built-in — fast ternary select
- `saturate()`, `abs()`, `negate` — free instruction modifiers on A8+
- `[[function_constant]]` — compile-time constants enabling fast integer division
- `MTLComputePipelineState.maxTotalThreadsPerThreadgroup` **[NEW]** — queries occupancy based on register/threadgroup memory usage
- Vector memory types: `float2`, `float4`, `half2`, `half4` — enable vector loads on scalar GPU
- `half`, `short`, `ushort` — 16-bit types preferred for registers and arithmetic
- Auto-vectorization of neighboring struct field loads (iOS 10 compiler optimization) **[NEW]**

## Code Highlights

Passing a constant-address-space struct by reference to enable constant buffer preloading:
```metal
struct LightData {
    Light lights[MAX_LIGHTS];
    uint  lightCount;
};

fragment float4 myFragment(constant LightData &lightData [[buffer(0)]], ...) {
    for (uint i = 0; i < lightData.lightCount; i++) { ... }
}
```

Using `simdgroup_barrier` instead of `threadgroup_barrier` for a SIMD-sized thread group:
```metal
// Thread group fits within one SIMD group — use cheaper barrier
simdgroup_barrier(mem_flags::mem_threadgroup);
// instead of:
// threadgroup_barrier(mem_flags::mem_threadgroup);
```

Using typed literals to keep arithmetic in `half`:
```metal
half value = someHalf * 2.0h;   // half operation (fast)
// NOT: half value = someHalf * 2.0; // float operation (slow)
```

## Takeaways
- Choose `constant` address space for high-reuse read-only data (matrices, light structs) and pass by reference to unlock constant buffer preloading; use `device` for variable-size or per-vertex data.
- Avoid the two biggest pitfalls: dynamically indexed non-constant stack arrays and integer division by non-constant denominators—both can cause 30–100× slowdowns that dwarf all other optimizations.
- Use `half` for texture samples and interpolants, and `short`/`ushort` for indices and thread IDs to cut register pressure in half on A8+ GPUs.
- Cooperate with the compiler: write clear, idiomatic Metal (ternary selects, direct vector types, bounded constant buffers) rather than bit-trick micro-optimizations that obscure intent.

---
_Source: WWDC16 Session 606 page (abstract, transcript, and resource links)._
