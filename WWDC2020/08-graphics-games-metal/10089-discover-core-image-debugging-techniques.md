# Discover Core Image Debugging Techniques
**WWDC20 · Session 10089** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10089/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session introduces `CI_PRINT_TREE`, a powerful Core Image debugging environment variable that lets developers visualize the internal rendering graph that Core Image builds and optimizes for each render. The feature is built on the same infrastructure as Core Image's Xcode Quick Look support, so developers familiar with hovering over `CIImage` variables in the debugger will find it immediately recognizable.

The session covers how to enable `CI_PRINT_TREE` via Xcode scheme environment variables or Terminal, how to control its output (graph type, format, and additional options), how to retrieve the generated documents from macOS or iOS devices, and how to interpret the resulting PDF or PNG graphs to find color space issues, unexpected intermediate buffers, and performance bottlenecks.

## Key Topics

### What is CI_PRINT_TREE
- An environment variable that causes Core Image to dump render-graph visualizations at runtime.
- Built on the same Xcode Quick Look infrastructure: hovering over a `CIImage` in the debugger shows the image's recipe graph; `CI_PRINT_TREE` automates this for every render.
- Green nodes in graphs = warp kernels; red nodes = color kernels.
- Inputs at the bottom, output at the top.

### Enabling CI_PRINT_TREE
- **Xcode scheme**: Edit scheme → Arguments → Environment Variables → add `CI_PRINT_TREE` with the desired value.
- **Terminal**: Set the variable before launching the app's executable: `CI_PRINT_TREE="<value>" ./MyApp`.

### CI_PRINT_TREE Value Format
The value is a string composed of: `<graph-type>[,<output-type>[,<options>]]`

**Graph types** (combinable by summing):
- `1` — Initial graph: shows the state of each render before optimization. Useful for seeing color space assignments.
- `2` — Optimized graph: shows how Core Image reorders, combines, and prunes render stages.
- `4` — Concatenated graph: shows how Core Image assembles stages into GPU programs and identifies intermediate buffers.
- `7` — All three (1+2+4).

**Output types**:
- `pdf` — saves a PDF document per render.
- `png` — saves a PNG document per render.
- (none) — prints a compact text-only format to standard output.
- `CI_LOG_FILE=oslog` — redirects text output to Console.app (convenient for iOS).

**Options**:
- `context==<name>` — limit output to a specific named `CIContext`.
- `frame-number=<n>` — log only the nth render of each context.
- `dump-inputs` — include input images in documents.
- `dump-intermediates` — include intermediate buffer images in concatenated-graph documents (type 4). Shows per-pass execution time, pixel count, and pixel format.
- `dump-output` — include the final output image in documents.

### Retrieving Files
- **macOS**: Documents saved to the temporary items directory (sandboxed apps have a unique temp dir).
- **iOS**: Documents saved to the app's documents directory (falls back to temp directory).
  - Enable `UIFileSharingEnabled` (`Application supports iTunes file sharing` = YES) in the app's Info.plist to access the documents directory via Finder's Files pane when the device is connected.

### Interpreting Graphs
- Each node shows its **ROI** (region of interest) — the area of each stage needed for the render.
- Look for `colormatch` nodes to see color space conversions and verify correct working-space handling (e.g., HLG → Core Image linear).
- With `dump-intermediates` on a type-4 graph: intermediate buffer images reveal where rendering errors were introduced; absent intermediates were served from cache.
- Execution time, pixel count, and pixel format annotations identify which passes consume the most time and memory.

## APIs & Frameworks

### Core Image
- `CIImage` — the lazy recipe object visualized by CI_PRINT_TREE and Xcode Quick Look
- `CIFilter` — applied to `CIImage` to build the graph
- `CIContext` — the rendering context; named contexts can be targeted by the `context==name` option
- `CIContext(options:)` — `kCIContextName` option to name a context

### Xcode / Environment Variables
- `CI_PRINT_TREE` — main environment variable
- `CI_LOG_FILE=oslog` — redirect CI text output to Console.app (os_log)
- `UIFileSharingEnabled` (`Application supports iTunes file sharing`) — Info.plist key for iOS document access via Finder

## Code Highlights
No code samples were included — `CI_PRINT_TREE` is a pure environment variable/tooling feature requiring no code changes. Example usage in Xcode scheme:

```
Environment Variable: CI_PRINT_TREE
Value: 7,pdf,dump-intermediates
```

This produces PDF graphs for all three pipeline stages with intermediate buffer images included for every render.

## Takeaways
- Set `CI_PRINT_TREE=7,pdf,dump-intermediates` in your Xcode scheme to get the most comprehensive view of every Core Image render — initial, optimized, and concatenated graphs, with intermediate images.
- Graph type `1` reveals unexpected color space conversions; type `4` with `dump-intermediates` reveals unintended intermediate buffers that indicate missed concatenation opportunities.
- On iOS, enable `UIFileSharingEnabled` to easily retrieve the generated PDF/PNG documents via Finder without any extra tooling.
- Use `context==<name>` and `frame-number=<n>` options to reduce noise when debugging specific contexts or frames.

---
_Source: WWDC20 Session 10089 page (abstract, transcript, and resource links)._
