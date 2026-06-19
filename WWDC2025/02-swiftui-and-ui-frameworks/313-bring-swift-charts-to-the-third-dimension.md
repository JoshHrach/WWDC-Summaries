# Bring Swift Charts to the Third Dimension
**WWDC25 · Session 313** · [Watch](https://developer.apple.com/videos/play/wwdc2025/313/)

_Platforms:_ iOS 26, macOS 26, visionOS 26

## Overview
Swift Charts gains a third dimension in iOS 26, macOS 26, and visionOS 26 with the introduction of `Chart3D` — a new view that renders interactive three-axis charts using RealityKit under the hood. The session walks through the full Chart3D API: adding existing mark types in 3D space, introducing `SurfacePlot` for continuous functions, configuring axes and camera, and applying foreground styles. On visionOS the chart is a fully manipulable 3D object; on iOS and macOS it is an interactive 2D projection with gesture-driven orbit.

## Key Topics

### Chart3D View
`Chart3D` is the primary entry point. It takes a `content` closure similar to `Chart`, but mark positions are specified across three axes (x, y, z). The view wraps RealityKit rendering internally, providing hardware-accelerated 3D at full frame rate. On visionOS it appears as a floating volume; on iOS and macOS the user can drag to orbit the chart.

### Existing Marks in 3D
`PointMark`, `RuleMark`, and `RectangleMark` all work inside `Chart3D` with a third axis binding. Developers add a `z:` parameter to position marks in the new dimension. Existing foreground style, symbol, and size modifiers continue to work.

### SurfacePlot
`SurfacePlot(x:y:z:)` renders a continuous parametric surface. The trailing closure receives (x, z) coordinate values and returns the corresponding y value (or nil for undefined regions). This is ideal for machine learning regression surfaces, mathematical visualizations, and heat maps.

### Axes and Scale
New axis modifiers mirror their 2D counterparts:
- `chartZAxisLabel` — label for the z axis
- `chartZScale(domain:range:)` — domain and range for the z axis
- `chartZAxis` — customize z axis tick marks and labels

### Camera and Pose Control
`.chart3DPose(_:)` sets the initial viewing angle. The `Chart3DPose` type provides `.default` and `.front` presets, plus a custom initializer accepting azimuth and inclination angles. `.chart3DCameraProjection(_:)` switches between `.orthographic` (default) and `.perspective`.

### Foreground Styles for Surfaces
`SurfacePlot` supports `.heightBased` and `.normalBased` foreground styles for automatic color mapping along height or surface-normal direction respectively. Standard gradient types (`LinearGradient`, `EllipticalGradient`) are also supported.

## APIs & Frameworks

**Swift Charts (iOS 26, macOS 26, visionOS 26)**
- **[NEW]** `Chart3D` — 3D chart view backed by RealityKit
- **[NEW]** `SurfacePlot(x:y:z:)` — continuous 3D surface from a closure
- **[NEW]** `chartZAxisLabel` modifier — label the z axis
- **[NEW]** `chartZScale(domain:range:)` modifier — configure z axis scale
- **[NEW]** `chartZAxis` modifier — customize z axis appearance
- **[NEW]** `.chart3DPose(_:)` modifier — set viewing angle
- **[NEW]** `Chart3DPose` — `.default`, `.front`, custom azimuth/inclination
- **[NEW]** `.chart3DCameraProjection(_:)` modifier — `.orthographic`, `.perspective`
- **[NEW]** `.heightBased` foreground style — color by y value
- **[NEW]** `.normalBased` foreground style — color by surface normal direction
- `PointMark`, `RuleMark`, `RectangleMark` — updated to support z axis in Chart3D
- `EllipticalGradient`, `LinearGradient` — supported as foreground styles for SurfacePlot

**CreateML (used in session sample)**
- `MLLinearRegressor` — fit a regression model; surface coefficients used to drive `SurfacePlot`

## Code Highlights
A basic 3D scatter chart:
```swift
Chart3D(data) { item in
    PointMark(
        x: .value("Width", item.width),
        y: .value("Height", item.height),
        z: .value("Depth", item.depth)
    )
}
```

A regression surface plot:
```swift
Chart3D {
    SurfacePlot(x: "Flipper Length", y: "Body Mass", z: "Bill Length") { x, z in
        // Return model prediction for (x, z) input
        model.predict(flipperLength: x, billLength: z)
    }
    .foregroundStyle(.heightBased)
}
.chart3DPose(.default)
.chart3DCameraProjection(.perspective)
```

## Takeaways
- Use `Chart3D` to graduate existing 2D charts to 3D; most mark types and modifiers carry over.
- `SurfacePlot` is the key new mark type — ideal for machine learning model visualization and mathematical function plotting.
- Configure `.chart3DPose(_:)` and `.chart3DCameraProjection(_:)` to set the initial perspective; users can still interact to re-orient.
- Apply `.heightBased` or `.normalBased` foreground styles to SurfacePlot for perceptually informative color mapping without writing shader code.

---
_Source: WWDC25 Session 313 page (abstract, chapter summaries, code samples, and resource links)._
