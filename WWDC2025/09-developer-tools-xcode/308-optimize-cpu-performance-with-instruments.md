# Optimize CPU performance with Instruments
**WWDC25 · Session 308** · [Watch](https://developer.apple.com/videos/play/wwdc2025/308/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26

## Overview
This session introduces new hardware-assisted profiling capabilities in Instruments powered by Apple Silicon's performance monitoring units (PMUs). The new tools expose hardware counters — cache miss rates, branch mispredictions, instruction throughput, memory bandwidth — directly in the Instruments timeline alongside existing CPU Profiler data, enabling developers to diagnose bottlenecks that are invisible to sampling-based profilers.

The session is structured around a progression from "the code is slow" to "here is the exact hardware bottleneck and how to fix it," using a compute-intensive sample app to demonstrate each tool. The key insight is that CPU time alone does not explain why code is slow — hardware counter data reveals whether the bottleneck is memory-bound, compute-bound, or control-flow-bound.

## Key Topics

### Hardware Performance Counters
Apple Silicon CPUs expose a rich set of performance monitoring unit counters accessible without root privileges. Instruments now surfaces these as a dedicated instrument track that correlates with the existing CPU usage and time profiler data. Counters include: retired instructions, CPU cycles, cache references and misses (L1, L2, LLC), branch instructions and mispredictions, and memory load/store counts.

### The CPU Profiler and Counter Correlation
By pinning a PMU counter instrument alongside the standard CPU Profiler instrument, developers can identify call-tree nodes that exhibit high cache miss rates — narrowing from "this function is slow" to "this function is slow because it misses the L2 cache on every iteration."

### Memory-Bound vs. Compute-Bound Diagnosis
The session introduces a decision framework: compute the arithmetic intensity (instructions per byte transferred from memory) to classify code as memory-bound or compute-bound. Different optimization strategies apply:
- **Memory-bound**: improve data layout (AoS to SoA), reduce working set, prefetch
- **Compute-bound**: reduce instruction count, use SIMD intrinsics, unroll loops

### Branch Misprediction
The branch misprediction counter exposes costly speculation failures. The session demonstrates how replacing branch-heavy conditional logic with predicated (branchless) code or lookup tables eliminates misprediction stalls in tight loops.

### Sustained Execution and Thermal Correlation
New thermal state timeline data in Instruments shows when the device enters throttled thermal states during a profiling session. Correlating thermal events with performance counter data helps distinguish between algorithmic inefficiency and thermal throttling — two problems with very different solutions.

### Practical Workflow
1. Profile with CPU Profiler to find the hot path.
2. Add a PMU counter instrument for cache misses.
3. Identify the specific function with the highest miss rate.
4. Inspect memory access patterns in the source.
5. Apply data layout or algorithm changes.
6. Re-profile to confirm improvement.

## APIs & Frameworks

- **Instruments** — extended with hardware PMU counter instruments **[NEW]**
  - L1/L2/LLC cache miss rate tracks
  - Branch misprediction counter track
  - Retired instructions and cycle count tracks
  - Memory bandwidth track
- **CPU Profiler instrument** (existing) — sampling-based call tree, now correlates with PMU data
- **Thermal state timeline** **[NEW in Instruments]** — device thermal events overlaid on profile
- **Metal Performance HUD** (mentioned) — GPU-side complement to CPU profiling
- **os_signpost** (existing) — custom intervals that align with hardware counter tracks

## Code Highlights

```swift
// Mark a critical section with os_signpost for correlation in Instruments
import os.signpost

let log = OSLog(subsystem: "com.example.app", category: .pointsOfInterest)

os_signpost(.begin, log: log, name: "ProcessBatch")
processBatch(data)
os_signpost(.end, log: log, name: "ProcessBatch")
```

```swift
// Data layout change: Array of Structs → Struct of Arrays (reduces cache misses)
// Before (AoS - poor spatial locality for single-field access)
struct Particle { var x, y, z: Float; var mass: Float }
var particles: [Particle]

// After (SoA - sequential access pattern matches cache line width)
struct ParticleBuffer {
    var x: [Float]; var y: [Float]; var z: [Float]; var mass: [Float]
}
```

## Takeaways

- CPU time is a symptom; hardware counters reveal the cause — always check cache miss rate before restructuring algorithms.
- The memory-bound vs. compute-bound classification drives the right optimization strategy; applying compute optimizations to memory-bound code yields little improvement.
- `os_signpost` intervals align perfectly with hardware counter tracks in Instruments — mark batch boundaries to narrow counter data to specific operations.
- Thermal throttling is a performance issue with a non-algorithmic fix; identify it via the thermal timeline before optimizing code that is already efficient.

---
_Source: WWDC25 Session 308 page (abstract, chapter summaries, code samples, and resource links)._
