# Compose Advanced Graphics Effects with SwiftUI
**WWDC26 · Session 322** · [Watch](https://developer.apple.com/videos/play/wwdc2026/322/)

_Platforms:_ iOS, iPadOS, macOS, visionOS

## Overview
This session introduces a "creative pipeline" mental model for building sophisticated visual effects in SwiftUI: each stage takes data as input, transforms it, and passes the result to the next stage. The running example is a podcast app that transforms cover art into an animated background visualizer, syncs a transcript view to playback time, and places floating timestamps using alignment guides — all without leaving SwiftUI.

The three technical pillars are: Metal layer-effect shaders (GPU-side per-pixel transformations applied via `.layerEffect`), `TimelineView` for frame-by-frame animation driven by elapsed time, and `alignmentGuide` for semantically anchoring floating overlays to their host view without manual offsets.

A companion sample project ("Composing advanced graphics effects with SwiftUI") is available for download.

## Key Topics

### Design Breakdown (1:40)
The session starts by decomposing the finished podcast UI into pipeline stages: cover-art image → blur → warp shader → animated time input → foreground transcript. This decomposition approach applies to any complex design — identify each transformation and chain simple primitives.

### Cover Art and Shader Effects (4:11)
Three SwiftUI shader modifier types are introduced:
- **Color effects** — map each pixel's color to a new color (no sampling of neighbors)
- **Distortion effects** — remap where each pixel samples from (no color math)
- **Layer effects** — full access to the layer texture; can sample any position

The session builds a layer-effect "backgroundWarp" shader that reads a noise texture (`texture2d<half>`) and uses domain warping (sampling the noise twice) to produce organic, non-repeating offsets. The Metal shader is declared with `[[stitchable]]` and accessed via `ShaderLibrary.backgroundWarp(...)`.

### Driving Animation with Time (11:07)
Shaders are stateless; to animate them, time must come from outside. `TimelineView(.animation)` fires every frame with a `Date`. The elapsed interval since a `@State var startDate` is passed as a `.float(elapsed)` argument to the shader, making the warp pattern flow continuously.

### Time-Synced Transcript View (12:00)
A `LazyVStack` of `Text` views inside `ScrollViewReader` + `ScrollView`. A `PlaybackState` observable drives the current line index. `onChange(of: playback.currentLineIndex)` calls `scrollProxy.scrollTo(_:anchor: .center)` to keep the active line visible. Inactive lines are visually faded via a `.transcriptLineStyle(isCurrent:)` custom modifier.

### Floating Timestamps with Alignment Guides (13:18)
SwiftUI alignment guides: two views sharing the same alignment point snap together. By overriding `.alignmentGuide(.bottom)` on the overlay timestamp to return `$0[.top]` (the view's own top edge), the timestamp floats above the baseline of the transcript line it annotates — no `offset` or `position` modifier needed.

## APIs & Frameworks

**SwiftUI**
- `Image.blur(radius:)` — Gaussian blur modifier
- `.layerEffect(_:maxSampleOffset:)` — **[NEW / primary use]** apply a Metal layer shader
- `.colorEffect(_:)` — Metal color-effect shader modifier
- `.distortionEffect(_:maxSampleOffset:)` — Metal distortion shader modifier
- `ShaderLibrary` — accesses `.metal` files compiled into the app bundle
- `Shader` — value type wrapping a Metal function + arguments
- `.float(_:)`, `.float2(_:)`, `.image(_:)` — shader argument constructors
- `GeometryReader` — provides `proxy.size` for passing dimensions to shaders
- `.ignoresSafeArea()` — used to extend the shader background edge-to-edge
- `TimelineView(.animation)` — **[key]** fires every display frame
- `TimelineView.Context.date` — current frame timestamp
- `ScrollViewReader` + `scrollProxy.scrollTo(_:anchor:)`
- `onChange(of:)` modifier
- `LazyVStack(alignment:spacing:)` inside `ScrollView`
- `Text` with custom view modifier for highlight/fade styling
- `.overlay(alignment:)` modifier
- `alignmentGuide(_:compute:)` — custom alignment override
- `Alignment`, `HorizontalAlignment`, `VerticalAlignment`
- `$0[.top]`, `$0[.bottom]` — dimension shortcuts in alignment guide closures
- `@State var startDate = Date.now`

**Metal / Shader**
- `[[stitchable]]` function attribute — required for SwiftUI shader functions
- `SwiftUI::Layer` — layer texture type for `.layerEffect`
- `layer.sample(position)` — sample the layer at a given coordinate
- `texture2d<half>` — noise/image texture parameter
- `constexpr sampler(address::repeat, filter::linear)` — texture sampler

## Code Highlights

Layer effect with noise-based domain warping:
```metal
[[stitchable]] half4 backgroundWarp(
    float2 position, SwiftUI::Layer layer,
    float2 size, texture2d<half> noiseTex, float time
) {
    constexpr sampler s(address::repeat, filter::linear);
    float2 uv = position / size + float2(time * 0.05);
    half4 n = noiseTex.sample(s, uv);
    float2 q = float2(n.r, n.g);
    n = noiseTex.sample(s, uv + q);   // domain warp
    float2 offset = (float2(n.r, n.g) - 0.5) * 200.0;
    return layer.sample(position + offset);
}
```

Driving the shader with `TimelineView`:
```swift
TimelineView(.animation) { timeline in
    let elapsed = timeline.date.timeIntervalSince(startDate)
    CoverArtView()
        .layerEffect(
            ShaderLibrary.backgroundWarp(.float2(proxy.size),
                                         .image(Image("NoiseTexture")),
                                         .float(elapsed)),
            maxSampleOffset: .zero)
}
```

Floating timestamp via alignment guide override:
```swift
Text(line.text)
    .overlay(alignment: .bottomLeading) {
        Text(line.formattedTimestamp)
            .alignmentGuide(.bottom) { $0[.top] }  // float above host view
    }
```

## Takeaways
- Use `.layerEffect` (not `.colorEffect` or `.distortionEffect`) when the shader needs to read neighboring pixels or composite multiple samples.
- Shaders are stateless; always feed time and dynamic values from `TimelineView` or `@State` as explicit shader arguments — never store mutable data in Metal.
- Prefer `alignmentGuide` over `.offset` or `.position` for overlay anchoring; it's semantic and adapts automatically as the host view resizes.
- Think of any complex effect as a pipeline: decompose it into independent stages and chain the outputs before writing code.

---
_Source: WWDC26 Session 322 page (abstract, chapter summaries, code samples, and resource links)._
