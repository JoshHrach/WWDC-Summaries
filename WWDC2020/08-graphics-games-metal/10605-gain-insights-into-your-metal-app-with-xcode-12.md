# Gain Insights into Your Metal App with Xcode 12
**WWDC20 · Session 10605** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10605/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session provides a practical tour of the Metal debugging and performance tools in Xcode 12, centered on the newly redesigned Metal Debugger Summary view and its Insights system. The session uses Larian Studios' Divinity: Original Sin 2 on iPad Pro as a real-world example, demonstrating how to capture a frame, read the new summary, and let the Insights panel guide optimization of load/store actions and GPU architecture-specific issues.

The Insights panel is the key new feature — it surfaces actionable, context-aware suggestions about memory usage, bandwidth, performance, and Metal API usage, each with a description, hint, and links to documentation. This makes the tooling accessible even to developers new to Apple GPU optimization.

Metal System Trace in Instruments gains a new Shader Timeline track and per-shader GPU time estimates. GPU counters are also now available for non-Apple GPUs.

## Key Topics

**Metal Debugger — New Summary View**
- Single consolidated view showing: encoder count, draw call count, frame time, vertex count, memory usage by resource category
- Insights panel **[NEW]** — categorized into Memory, Bandwidth, Performance, and Metal API sections
- Each insight includes: problem description, fix hint, links to documentation, and a "Show in Dependencies" button

**Dependency Viewer**
- Bird's-eye overview of all GPU passes and resource dependencies between encoders
- Identifies unused load/store actions (resources stored but never read downstream)
- Accessed via "Show Dependencies" button in summary or by clicking any command buffer/encoder
- Insight example: encoder storing depth/stencil textures that no downstream encoder reads → change store action to `.dontCare` → saves bandwidth (11 MB in demo)

**GPU Performance Counters**
- Accessed via "Counters" in Metal Debugger navigator
- Shows GPU time per pass/encoder across the frame
- GPU counters now available for non-Apple GPUs **[NEW]**

**Shader Profiler**
- "View Frame by Pipeline State" → select render pipeline → per-line GPU performance breakdown in shader source
- Supports live shader edits with immediate performance preview

**Shader Debugger**
- Access via the debug button in debug bar → select pixel (fragment), geometry (vertex), or thread (compute)
- Shows variable state over all threads for the entire shader execution — no stepping required
- Supports live source edits with live results

**Memory Viewer**
- Shows all allocated resources, their sizes, types, volatility, and usage status
- Filter by resource type, access pattern, volatility, or "unused" flag

**Metal System Trace (Instruments)**
- Encoder Timeline: GPU track showing command buffer timeline, color-coded by frame; identifies GPU idle bubbles
- Shader Timeline **[NEW]**: shows which shaders ran at sampled times within each encoder; approximate per-shader GPU time in table view
- Performance Limiter tracks: GPU counter values over time (enable in Metal Application recording options → GPU Counter Set)
- GPU counters now for non-Apple GPUs **[NEW]**
- Memory Allocations track: resource alloc/dealloc events for detecting memory leaks

**Apple GPU Architecture Insight**
- Apple GPUs use Tile Based Deferred Rendering (TBDR) with automatic Hidden Surface Removal (HSR)
- Depth pre-pass (used on non-Apple GPUs to reduce overdraw) is unnecessary on Apple silicon — HSR does this automatically
- Insight example: identified a 1.71 ms depth pre-pass that was rendering the full scene into depth/stencil before the main pass — safe to eliminate entirely on Apple GPUs

## APIs & Frameworks

### Metal Debugger (Xcode 12)
- GPU Frame Capture — enable in scheme options → use camera icon in debug bar to capture
- Summary View **[NEW]** — consolidated frame metrics and Insights panel
- Insights **[NEW]** — actionable suggestions for memory, bandwidth, performance, Metal API
- Dependency Viewer — resource dependency graph between encoders
- GPU Performance Counters — per-pass timing detail
- Shader Profiler — per-line shader performance with live edit
- Shader Debugger — full-execution variable state, live edit
- Memory Viewer — resource inventory with filtering
- Shader Validation **[NEW]** — additional correctness checking (covered in companion session)

### Metal API
- `MTLRenderPassDescriptor.depthAttachment.storeAction` — set to `.dontCare` when depth is not read by subsequent encoders
- `MTLRenderPassDescriptor.stencilAttachment.storeAction` — same as above
- `MTLStoreAction.dontCare` — avoids unnecessary memory write bandwidth
- `MTLStoreAction.store` — retains data for subsequent encoder reads

### Instruments — Metal System Trace
- Encoder Timeline track — command buffer and encoder activity over time
- Shader Timeline track **[NEW]** — per-shader GPU sample data within encoders
- Performance Limiter tracks — GPU bottleneck counters over time
- Memory Allocations track — resource lifecycle events

## Code Highlights

No code snippets in this session — the content is entirely tooling-focused. Key workflow:

1. Enable GPU Frame Capture in scheme options
2. Run Metal app on device, navigate to frame of interest
3. Click camera icon in debug bar → opens Metal Debugger
4. Read Summary view → check Insights for categorized recommendations
5. Click "Show in Dependencies" on a bandwidth insight → Dependency Viewer shows which encoders need store action changes
6. Use GPU Counters to measure time per encoder, Shader Profiler for per-line costs

## Takeaways
- The new Insights panel in the Metal Debugger Summary view provides expert-level guidance without requiring deep Apple GPU knowledge — use it as the entry point for every optimization session.
- Changing unnecessary `.store` actions to `.dontCare` is one of the highest-impact, lowest-effort bandwidth optimizations on Apple GPUs — the Dependency Viewer makes it trivial to find these.
- Depth pre-passes are counterproductive on Apple GPUs due to TBDR + HSR — the Insights panel will flag these automatically with a recommendation to remove them.
- The new Shader Timeline in Metal System Trace makes it possible to pinpoint exactly which shaders are consuming GPU time within an encoder, moving beyond aggregate encoder time to per-shader accountability.

---
_Source: WWDC20 Session 10605 page (abstract, chapter summaries, code samples, and resource links)._
