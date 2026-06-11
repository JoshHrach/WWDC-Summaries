# Design immersive environments for visionOS apps and the spatial web
**WWDC26 · Session 234** · [Watch](https://developer.apple.com/videos/play/wwdc2026/234/)

_Platforms:_ visionOS 27

## Overview
This session is a design-focused guide for creating photorealistic immersive environments that work in visionOS apps, websites, and SharePlay experiences. It walks through the full production pipeline in three stages: pre-production (concept and location scouting), production (on-location photography and capture), and post-production (panorama processing, 3D asset creation, and real-time effects).

The session emphasizes the distinct nature of immersive environments compared to panoramas or flat media — the environment must feel like a real place the viewer inhabits. Intentional design choices in composition, framing, and what to omit are as important as technical fidelity. The presenter uses a natural landscape environment as the running example throughout, sharing professional-grade tips for both solo creators and teams with advanced resources.

Post-production covers photogrammetry-derived sources for 3D assets, how to maintain photorealism in real-time geometry, cleanup of source imagery, and critical finishing touches: adding natural motion (e.g., swaying foliage, water) and Spatial Audio to sell the sense of presence.

## Key Topics

### Pre-production
- Define the mood, purpose, and audience before any capture begins; ask what story the environment tells.
- Location scouting: visit candidate sites in person, photograph lighting conditions at multiple times of day, identify elements to keep or remove (power lines, signage, people).
- Plan for the specific field-of-view and scale of Apple Vision Pro — what works for a flat image may not work when the viewer is inside the scene.

### Production
- High-quality primary source capture: use a high-resolution panoramic camera rig or photogrammetry setup.
- Secondary photography: per-object reference shots for albedo, normals, and reflection at multiple exposure levels.
- Capture tips for solo creators: bracketing exposures, HDR merges, tethered shooting for consistent framing.
- For teams: structured capture of multiple camera positions, light probes, and video references for motion.

### Post-production
- Processing source imagery into a seamless high-resolution panorama; removing unwanted elements via inpainting.
- Building 3D geometry from reference photographs; keeping polygon and texture budgets practical for real-time rendering.
- Motion: subtle particle systems, skeletal animation, and texture scrolling to animate natural elements like leaves, water, clouds.
- Spatial Audio: ambient bed, point sources, and reverb zones tied to environment geometry; matching perceived distance and direction to the visual.
- Integration paths: exporting assets in USD/USDZ for use in Reality Composer Pro or visionOS app immersive space; using the `<model>` element with `environmentmap` for the spatial web.

### Design Principles
- Make intentional composition decisions; the viewer cannot be directed, so the scene must be compelling in every direction.
- Avoid repeating tile patterns, overly clean CG surfaces, or physics-defying details that break immersion.
- Sound and motion are not optional additions — they are the features that most strongly produce a sense of presence.

## APIs & Frameworks

### RealityKit / visionOS
- Immersive Space scene type for rendering full-surround environments
- `ShaderGraphMaterial` — used for animated environment materials (scrolling textures, procedural motion)
- `SpatialAudioComponent` / reverb zones anchored to environment geometry
- USDZ as the environment asset format; exported from DCC tools and loaded at runtime

### Spatial Web (Safari / visionOS)
- `<model src="..." environmentmap="...">` HTML element: inline 3D model with HDR lighting **[NEW]**
- `requestImmersive()` JavaScript API: transitions the model to a full environment **[NEW]**
- `:immersive` CSS pseudo-class for layout changes during immersive mode **[NEW]**

### Toolchain (Not APIs, but referenced)
- Reality Composer Pro 3 — scene assembly, lighting, and Animation Graph for motion effects
- Photogrammetry pipeline (any DCC) → USD → USDZ export
- `usdcrush` command-line tool for mesh and texture compression before shipping

## Code Highlights

Declare a model with environment lighting on the web:
```html
<model src="landscape.usdz"
       environmentmap="landscape-lighting.hdr">
</model>
```

Transition to immersive from a button:
```javascript
enterButton.addEventListener("click", async () => {
    await model.requestImmersive();
});
```

## Takeaways
- Great immersive environments succeed through intentional design decisions in pre-production — knowing what to include and what to leave out matters more than raw resolution.
- Sound and motion are essential; a photorealistic static scene without ambient audio and natural movement will feel hollow.
- The same USDZ asset pipeline works for both native visionOS apps (via RealityKit/Reality Composer Pro) and the spatial web (via the `<model>` element).
- A phased approach — pre-production concept, on-location capture, and post-production assembly — yields more predictable results than improvising on set.

---
_Source: WWDC26 Session 234 page (abstract, chapter summaries, and resource links)._
