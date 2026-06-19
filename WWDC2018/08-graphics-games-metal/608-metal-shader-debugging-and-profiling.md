# Metal Shader Debugging and Profiling
**WWDC18 · Session 608** · [Watch](https://developer.apple.com/videos/play/wwdc2018/608/)

_Platforms:_ iOS 12, macOS Mojave 10.14, tvOS 12

## Overview
This session introduces three major new tools added to the Metal Frame Debugger in Xcode 10 for diagnosing and optimizing Metal shaders: the Geometry Viewer, the Shader Debugger, and an enhanced Shader Profiler with A11-specific instruction category data. Together they provide a complete loop of capture → inspect geometry → step through shader execution → profile per-line GPU cost.

The Geometry Viewer fills the gap between the existing input attribute table and the 3D scene by visualizing post-transform vertex data in a free-fly 3D view. The Shader Debugger brings CPU-style debugging to massively parallel GPU workloads, showing variable values for the selected thread and thousands of neighboring threads simultaneously. The enhanced Shader Profiler adds per-line instruction category breakdowns (ALU, memory, synchronization) for devices running the A11 Bionic chip.

## Key Topics

### Geometry Viewer
- Visualizes post-transform vertex output in a 3D view with a free-fly camera
- Shows per-vertex input attributes, indices, and output positions correlated in space
- Detects and flags degenerate triangles, out-of-frustum geometry, and NaN/Infinite position values
- Available per draw call alongside attachments and bound resources in the Metal Frame Debugger

### Shader Debugger
- Fully parallel GPU debugger: shows real data from the GPU, not an emulator
- Supports vertex, fragment, and compute shaders
- Entry points: click pixel (fragment), select vertex in Geometry Viewer (vertex), or enter thread ID (compute)
- Per-line variable values displayed inline in the source margin without requiring breakpoints
- NaN and Infinity values highlighted automatically
- Detail views show the variable value across the full primitive (vertex), rectangular pixel region (fragment), or thread group (compute)
- Debug Navigator provides a linear execution history with backward stepping via arrow keys
- Loops appear as expandable nodes; each iteration's variable values are accessible
- Thread switching: select any neighboring thread in the detail view to re-execute and compare
- Divergence mask shows which threads executed the same source line (useful for conditional branches)
- "Reload Shaders" button applies shader edits and reruns the captured frame immediately

### Shader Profiler
- Pipeline list sorted by GPU execution time; drill down to per-draw or per-pipeline timing
- Per-line execution cost for iOS and tvOS targets
- **A11-specific enhancements** **[NEW]**: per-line instruction category breakdown — ALU, memory reads (texture/buffer), synchronization/latency stalls
- Inline function cost attribution — navigate directly to the most expensive called function
- Workflow: identify expensive pass → inspect per-line cost → edit shader → reload → re-profile in one flow

### Offline Shader Compilation and Source Embedding
- New Metal compiler flag `-MO` (or Xcode Build Setting "Produce Debugging Information") embeds shader source into the `.metallib` for tool access
- Enable only in debug builds; do not ship source with production `.metallib`

## APIs & Frameworks

**Metal Frame Debugger (Xcode 10)**
- **Geometry Viewer** **[NEW]** — 3D post-transform vertex visualization per draw call; degenerate/NaN detection
- **Shader Debugger** **[NEW]** — line-by-line GPU shader debugger for vertex, fragment, and compute shaders; real GPU data
- Detail views for cross-thread variable inspection **[NEW]**
- Divergence mask visualization **[NEW]**
- "Reload Shaders" during debug session **[NEW]**
- Dependency Viewer / Dependency Graph **[NEW]** (covered in Session 612)
- Pipeline Statistics view — compile metrics, instruction type counts per shader
- GPU Counters — high-level performance counters per pass
- Shader Profiler — per-line GPU timing
- **A11 per-line instruction categories** **[NEW]**: ALU, memory, synchronization breakdown
- Inline function cost attribution **[NEW]**

**Metal Shading Language / Compiler**
- `metal` compiler flag `-MO` **[NEW]** — embed shader source into compiled `.metallib`
- Xcode Build Setting: "Produce Debugging Information" for Metal shaders **[NEW]**
- `half` precision type — significantly more efficient than `float` on A11 Bionic GPU

**Metal System Trace (Instruments)**
- GPU timeline — vertex, fragment, compute tracks
- Used as first-stop profiling before diving into Metal Frame Debugger

## Code Highlights

Identifying a dependent texture read causing synchronization stalls (before fix):
```metal
// Noise texture read → dependent offset → color texture read: GPU stalls waiting on noise sample
float2 offset = noiseTexture.sample(s, uv).xy;
float4 color  = colorTexture.sample(s, uv + offset);
```

Fix — replace dependent texture read with inline computation:
```metal
// Compute noise analytically; eliminates dependent read stall
float2 offset = computeNoise(uv);
float4 color  = colorTexture.sample(s, uv + offset);
```

Enabling shader source embedding (command line):
```sh
xcrun metal -MO -o MyShaders.metallib MyShaders.metal
```

## Takeaways
- Always check geometry correctness with the Geometry Viewer before debugging shaders — NaN positions and out-of-frustum geometry are invisible in the framebuffer but immediately visible in 3D.
- The Shader Debugger's detail views make it possible to distinguish good versus bad shader values across thousands of threads without reading the full source — divergence masks pinpoint conditional branch issues instantly.
- Dependent texture reads are a leading cause of synchronization stalls on A11; replacing them with analytical computation can reduce shader time by 5-10x.
- Embed shader sources into `.metallib` via `-MO` only in debug builds to maintain full tooling access without shipping intellectual property.

---
_Source: WWDC18 Session 608 page (abstract, chapter summaries, code samples, and resource links)._
