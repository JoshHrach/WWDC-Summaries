# Scale compute workloads across Apple GPUs
**WWDC22 · Session 10159** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10159/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
This session from Apple's GPU Software Engineering team provides a systematic framework for making Metal compute workloads scale linearly across the M1 GPU family — from an 8-core iPad GPU to a 64-core Mac Studio GPU. Linear scaling requires eliminating three classes of problems: inadequate work distribution that leaves GPU cores idle, GPU timeline gaps caused by CPU/GPU synchronization, and atomic contention that serializes global memory writes. A fourth topic covers tuning spatial and temporal memory access patterns to reduce Last Level Cache and MMU limiter pressure.

The session uses real-world examples including Affinity Photo, DaVinci Resolve, and Blender Cycles to illustrate concepts and quantify improvements.

## Key Topics

### GPU Scalability Concepts
- **Ideal scaling**: performance improvement is linearly proportional to the number of GPU cores.
- Two failure modes: plateau (GPU not saturated, idle gaps in timeline) and non-uniform scaling (GPU limiters hit at certain core counts).
- Bottlenecks shift when scaling up — optimize iteratively; expect to bounce between compute-bound and bandwidth-bound states.

### 1. Improve Work Distribution
- A dispatch creates a 3D grid of **threadgroups**; threadgroups are distributed to GPU cores.
- Each threadgroup is split into **SIMD-groups** (waves/warps); all Apple GPUs have SIMD width of 32 (`MTLComputePipelineState.threadExecutionWidth`).
- Max threads per threadgroup: 1024; max threadgroup memory: 32 KB.
- **Saturation rule of thumb**: 1,000–2,000 concurrent threads per GPU core. Multiply by GPU core count to get minimum total threads per dispatch.
  - 8-core: ~8K–16K threads; 16-core: ~16K–32K; 32-core: ~32K–64K; 64-core: ~64K–128K.
- **Use smaller threadgroups**: larger threadgroups can cause uneven distribution across cores; use the smallest multiple of 32 (SIMD width) that maps well to the workload.
- Xcode 14 GPU capture: **Max Theoretical Occupancy** counter (new) in compiler statistics helps identify under-dispatching.

### 2. Eliminate GPU Timeline Gaps
- **`commandBuffer.waitUntilCompleted`** causes large CPU/GPU synchronization gaps; replace with `MTLSharedEvent` callbacks.
- **Pipelining**: if the algorithm produces data for a subsequent batch, encode one or more batches in advance before waiting, so the GPU always has work queued.
- **Multiple queues**: submit independent work on a second `MTLCommandQueue` to prevent one waiting submission from blocking the GPU.
- **Indirect Command Buffers (ICBs)**: for tightly coupled dependency chains, encode the next batch directly from a GPU kernel, eliminating CPU round-trips entirely.
- **Concurrent dispatches** (`MTLDispatchTypeConcurrent`): allows the driver to interleave independent dispatches from the same command encoder. Eliminates ramp-up/tail-end idle gaps between serial back-to-back kernels.
  - Requires manual barriers (`MTLComputeCommandEncoder.memoryBarrier(...)`) between dependent passes.
  - Example: processing two images concurrently improved throughput by 30%; three images in parallel improved by 70% compared to serial execution.

### 3. Atomics: Reduce Global Memory Contention
- Global atomics (coherent across entire GPU) create contention when many threads write the same address. More GPU cores = more contention.
- **SIMD-group instructions**: use `simd_prefix_exclusive_sum`, `simd_min`, and similar to reduce values within a SIMD-group entirely in registers — no memory round-trip.
- **Threadgroup atomics**: threadgroup memory is local to each GPU core; atomic adds within a threadgroup are fast and scale with core count.
- **Two-level reduction pattern**: (1) SIMD-group instruction reduces 32 threads to 1 value per SIMD-group; (2) the last thread per SIMD-group does a threadgroup atomic add to a single threadgroup memory value; (3) one thread per threadgroup does a single global atomic add. This scatters global atomics over time, drastically reducing contention.

### 4. Optimize GPU Limiters: Spatial and Temporal Memory Access
- High **MMU Limiter** and high **Last Level Cache Limiter** with low LLC utilization indicate excessive memory span (data spread across too much address space).
- Two tuning strategies:
  - **Rearrange data layout** to match the compute kernel's access pattern (e.g., tile/stripe a row-major buffer into vertical stripes to match a 2D threadgroup access pattern).
  - **Reshape the dispatch grid** (e.g., use a more rectangular threadgroup shape to align with row-major memory layout).
- **Blender Cycles example**: sorting ray hits by material type improved thread divergence but increased spatial memory divergence (high MMU limiter). Adding **memory range partitioning** before sorting confined each partition's indices to a sub-range of the full buffer, reducing MMU pressure 20%, increasing GPU read bandwidth significantly, and improving render performance 10–30% depending on scene.

### New Tools in Xcode 14 and Instruments
- **Max Theoretical Occupancy** (Xcode 14) — new counter in compiler statistics; compares theoretical vs. actual occupancy.
- **MMU Limiter**, **MMU Utilization Counter**, **MMU TLB Miss Rate Counter** — new in Xcode GPU Frame Capture and Metal System Trace.
- Use "Top GPU Limiter" in Metal System Trace to drive iterative optimization.

## APIs & Frameworks

**Metal**
- `MTLComputePipelineState.threadExecutionWidth: Int` — SIMD-group width (always 32 on Apple GPUs)
- `MTLComputePipelineState.maxTotalThreadsPerThreadgroup: Int` — max threads per threadgroup (1024)
- `MTLComputeCommandEncoder.dispatchThreadgroups(_:threadsPerThreadgroup:)` — standard threadgroup dispatch
- `MTLComputeCommandEncoder.dispatchThreadgroupsWithIndirectBuffer(...)` — GPU-driven threadgroup dispatch
- `MTLDispatchType.concurrent` — **[reinforced]** allow driver to interleave independent dispatches
  - `MTLComputeCommandEncoder(dispatchType: .concurrent)`
- `MTLComputeCommandEncoder.memoryBarrier(resources:after:before:)` — explicit barrier between concurrent dispatches
- `MTLSharedEvent` — CPU/GPU synchronization with lower overhead than `waitUntilCompleted`
  - `MTLSharedEvent.notify(_:atValue:block:)` — callback-based completion handling
- `MTLCommandBuffer.waitUntilCompleted()` — avoid in performance-critical paths; replace with `MTLSharedEvent`
- Indirect Command Buffers (`MTLIndirectCommandBuffer`) — GPU-side encoding of subsequent passes

**Metal Shading Language (MSL)**
- `simd_prefix_exclusive_sum(x)` — SIMD-group prefix sum (no memory round-trip)
- `simd_min(x)` / `simd_max(x)` / `simd_sum(x)` — SIMD-group reduction operations
- Threadgroup memory atomics: `atomic_fetch_add_explicit(&tg_value, val, memory_order_relaxed)` targeting threadgroup address space
- `metal::simdgroup_barrier(mem_flags::mem_threadgroup)` — synchronize within SIMD-group

**Metal Performance Shaders (MPS) / MPSGraph**
- Use MPS/MPSGraph primitives when possible — Apple guarantees they are optimized for all hardware tiers; they free you from manual tuning of work distribution and memory layout.

**Instruments (Metal System Trace, Xcode GPU Frame Capture)**
- Top Performance Limiter — primary driver for optimization target selection
- LLC (Last Level Cache) Limiter / LLC Utilization
- MMU Limiter — **[NEW counter in Xcode 14]**
- MMU Utilization Counter — **[NEW]**
- MMU TLB Miss Rate Counter — **[NEW]**
- Max Theoretical Occupancy — **[NEW in Xcode 14 compiler statistics]**

## Code Highlights

Choosing appropriate threadgroup size (rule of thumb):
```swift
// Ensure total threads >= 1000-2000 per GPU core
// e.g., M1 Max 32-core: aim for ~32K-64K total threads minimum
let threadsPerThreadgroup = MTLSize(width: 32, height: 1, depth: 1) // smallest SIMD multiple
let threadgroupsPerGrid = MTLSize(width: (totalWork + 31) / 32, height: 1, depth: 1)
encoder.dispatchThreadgroups(threadgroupsPerGrid, threadsPerThreadgroup: threadsPerThreadgroup)
```

Replace waitUntilCompleted with MTLSharedEvent:
```swift
let event = device.makeSharedEvent()!
commandBuffer.encodeSignalEvent(event, value: 1)
commandBuffer.commit()
event.notify(MTLCreateSystemDefaultDevice()!.makeCommandQueue()!,
             atValue: 1) { _, _ in
    // Process results without blocking CPU
}
```

Concurrent dispatch for independent passes:
```swift
let encoder = commandBuffer.makeComputeCommandEncoder(dispatchType: .concurrent)!
encoder.setComputePipelineState(pipelineA)
encoder.dispatchThreadgroups(gridA, threadsPerThreadgroup: tgSize)
// Barrier only where needed
encoder.memoryBarrier(resources: [outputBufferA], after: .compute, before: .compute)
encoder.setComputePipelineState(pipelineB) // independent of A — runs concurrently
encoder.dispatchThreadgroups(gridB, threadsPerThreadgroup: tgSize)
encoder.endEncoding()
```

Two-level atomic reduction (MSL):
```metal
kernel void reduction(device float* input, device atomic_float* output,
                      threadgroup float* tg_sum [[threadgroup(0)]],
                      uint tid [[thread_position_in_threadgroup]],
                      uint simd_lid [[thread_index_in_simdgroup]],
                      uint simd_gid [[simdgroup_index_in_threadgroup]]) {
    float val = input[...];
    // Level 1: SIMD-group reduce (no memory round-trip)
    float simd_total = simd_sum(val);
    // Level 2: threadgroup atomic (one per SIMD-group, local to GPU core)
    if (simd_lid == 31) atomic_fetch_add_explicit(
        (threadgroup atomic_float*)&tg_sum[0], simd_total, memory_order_relaxed);
    threadgroup_barrier(mem_flags::mem_threadgroup);
    // Level 3: single global atomic per threadgroup
    if (tid == 0) atomic_fetch_add_explicit(output, tg_sum[0], memory_order_relaxed);
}
```

## Takeaways
- GPU scaling failures fall into three categories — under-saturation, timeline gaps, and atomic contention — each requiring different fixes; profile first to identify which applies.
- Ensure each dispatch produces at least 1,000–2,000 threads per GPU core; use smaller threadgroups (multiples of SIMD width 32) for better work balance across cores.
- Replace `waitUntilCompleted` with `MTLSharedEvent` callbacks and use concurrent dispatch (`MTLDispatchTypeConcurrent`) to eliminate GPU idle time between independent passes.
- Use SIMD-group instructions and threadgroup memory atomics to implement two-level reduction patterns, replacing high-contention global atomics with core-local parallelism that scales with GPU core count.
- Monitor the MMU Limiter (new in Xcode 14) and restructure data layouts or dispatch grids to improve memory locality — the Blender Cycles experiment delivered 10–30% performance improvement through memory range partitioning alone.

---
_Source: WWDC22 Session 10159 page (abstract, transcript, and resource links)._
