# Building Custom Views with SwiftUI
**WWDC19 · Session 237** · [Watch](https://developer.apple.com/videos/play/wwdc2019/237/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
This session dives below the surface of SwiftUI to explain how the layout system works under the hood and how to build high-performance custom graphics and animations. Presented by Dave Abrahams and John Harper, it covers two major subsystems — layout and graphics — then combines them in a live-coded demo of an interactive, animatable pie-chart control.

The layout half explains the three-step parent-child size negotiation protocol, how each view controls its own bounds, stack layout algorithms, layout priorities, and custom alignment guides. The graphics half covers `Shape`, fill styles (colors, gradients, images), `Path`, custom shapes conforming to the `Shape` protocol, view modifiers, custom transitions, and the `drawingGroup()` modifier for Metal-backed rendering of complex graphics trees.

## Key Topics

**Layout Protocol — Three Steps**
- Parent proposes a size → child chooses its own size → parent positions the child
- SwiftUI can never force a size on a child; the child always has final say
- Views at the top of a `body` chain are *layout neutral* — their bounds equal the bounds of their `body`
- `frame(width:height:)` is a view, not a constraint; its child may be smaller or larger than the frame

**Adaptive Modifiers**
- `padding()` without arguments uses platform/Dynamic Type-appropriate spacing
- `padding(.leading)` / `padding(.trailing)` for edge-specific adaptive padding
- Prefer adaptive modifiers early in development; add explicit values only when a spec demands them

**Stack Layout Algorithm**
- HStack/VStack subtract inter-child spacing first, then divide remaining space among children
- Children are sized in order of *least flexible* first (fixed > bounded > fully flexible)
- Each child's claimed size is deducted before the next child is offered its share
- `layoutPriority(_:)` — children with higher priority are offered space before lower-priority ones; lower-priority minimum widths are reserved first

**Custom Alignment Guides**
- Built-in alignments: `.top`, `.center`, `.bottom`, `.leading`, `.trailing`, `.firstTextBaseline`, `.lastTextBaseline`
- Custom alignment: define an `enum` conforming to `AlignmentID`, provide a default value, expose a `static let` on `VerticalAlignment` or `HorizontalAlignment`
- Use `.alignmentGuide(_:computeValue:)` on a child to project an alignment value through nested stacks to an outer container
- `lastTextBaseline` on non-text views defaults to the view's bottom edge; override via `alignmentGuide` to specify a visual baseline (e.g., 87.4% from the top of an image)

**Shapes and Fill Styles**
- `Shape` protocol — single requirement: `path(in rect: CGRect) -> Path`
- Shapes are views; all SwiftUI layout and animation modifiers apply to them
- `Circle()`, `Rectangle()`, `Capsule()`, `Ellipse()`, `RoundedRectangle(cornerRadius:)` — built-in
- `.fill(_:)` — fills shape with a color or gradient, returning a `View`
- `.stroke(_:lineWidth:)` — strokes on the shape's path
- `.strokeBorder(_:lineWidth:)` — strokes inside the shape's border

**Gradients**
- `Gradient(colors:)` — one-dimensional color ramp, equally-spaced stops
- `Gradient(stops:)` — custom stop positions
- `LinearGradient(gradient:startPoint:endPoint:)` — linear
- `RadialGradient(gradient:center:startRadius:endRadius:)` — radial
- `AngularGradient(gradient:center:angle:)` — conical/conic gradient

**Custom Shapes with `animatableData`**
- Conform to `Shape` + declare `var animatableData: AnimatablePair<…>` (or any `VectorArithmetic`)
- SwiftUI interpolates `animatableData` between old and new values when the shape changes
- Delegate to `AnimatablePair` of the shape's geometric properties to compose multi-property animation

**View Modifiers and Custom Transitions**
- `ViewModifier` protocol — `body(content:)` transforms one view into another
- Use modifiers to package reusable styling or behavioral transforms
- `AnyTransition.modifier(active:identity:)` — define a custom insertion/removal transition from two `ViewModifier` values
- Symmetric transitions: use `.scale`, `.opacity`, or custom view modifiers as both active and identity states

**`drawingGroup()` — Metal-Accelerated Rendering**
- Flattens a SwiftUI view subtree into a single `CAMetalLayer`-backed `UIView` / `NSView`
- All drawing (shapes, text, images) is composited in Metal GPU in a single pass
- Ideal for large numbers of graphic elements; should not be applied to regular control-heavy views
- Behavior is identical to standard rendering but performance scales much better with element count

**ZStack**
- Layers children depth-wise (into the screen) rather than horizontally or vertically
- Default alignment: `.center`; can specify any 2D alignment
- Used to composite multiple `Shape` views into a single diagram

## APIs & Frameworks

### SwiftUI Layout (NEW)
- `View.frame(width:height:alignment:)` **[NEW]** — positions content in a fixed-size frame
- `View.frame(minWidth:idealWidth:maxWidth:minHeight:idealHeight:maxHeight:alignment:)` **[NEW]** — flexible frame
- `View.padding(_:_:)` **[NEW]** — adaptive or explicit padding
- `View.layoutPriority(_:)` **[NEW]** — controls which children are sized first in stacks
- `View.alignmentGuide(_:computeValue:)` **[NEW]** — custom alignment value for a specific alignment
- `AlignmentID` protocol **[NEW]** — implement to create a custom alignment
- `VerticalAlignment(AlignmentID.Type)` / `HorizontalAlignment(AlignmentID.Type)` **[NEW]** — instantiate custom alignments
- `HStack(alignment:spacing:)` / `VStack(alignment:spacing:)` / `ZStack(alignment:)` **[NEW]**
- `Spacer()` **[NEW]** — flexible space that expands to fill available room in a stack

### SwiftUI Graphics (NEW)
- `Shape` protocol **[NEW]** — `path(in:) -> Path` requirement; shapes are views
- `Path` **[NEW]** — Bézier path builder: `addArc`, `addLine`, `addCurve`, `closeSubpath`
- `Circle()`, `Rectangle()`, `Capsule()`, `Ellipse()`, `RoundedRectangle(cornerRadius:)` **[NEW]**
- `View.fill(_:style:)` **[NEW]** — fills a shape with a `ShapeStyle`
- `View.stroke(_:lineWidth:)` **[NEW]** — strokes a shape
- `View.strokeBorder(_:lineWidth:)` **[NEW]** — inset stroke
- `Color` **[NEW]** — conforms to `ShapeStyle` and `View`
- `Gradient(colors:)` / `Gradient(stops:)` **[NEW]**
- `LinearGradient` **[NEW]**
- `RadialGradient` **[NEW]**
- `AngularGradient` **[NEW]**
- `AnimatableData` / `VectorArithmetic` — custom shape animation
- `AnimatablePair<First, Second>` **[NEW]** — compose two animatable values
- `ViewModifier` protocol **[NEW]** — `func body(content: Content) -> some View`
- `AnyTransition.modifier(active:identity:)` **[NEW]** — custom insertion/removal transition
- `View.transition(_:)` **[NEW]** — attach a transition to a view
- `View.drawingGroup(opaque:colorMode:)` **[NEW]** — Metal-backed composite rendering
- `View.shadow(color:radius:x:y:)` **[NEW]** — drop shadow
- `View.blur(radius:opaque:)` **[NEW]** — Gaussian blur
- `View.opacity(_:)` **[NEW]** — transparency
- `View.scaleEffect(_:anchor:)` **[NEW]** — scale transform
- `View.rotationEffect(_:anchor:)` **[NEW]** — 2D rotation
- `View.onTapGesture(count:perform:)` **[NEW]** — tap handler
- `withAnimation(_:_:)` **[NEW]** — explicit animation scope
- `ForEach` **[NEW]** — generate views from a collection (used with `ZStack` for multi-shape diagrams)

## Code Highlights

Custom shape with `animatableData`:
```swift
struct WedgeShape: Shape {
    var wedge: WedgeGeometry
    var animatableData: WedgeGeometry.AnimatableData {
        get { wedge.animatableData }
        set { wedge.animatableData = newValue }
    }
    func path(in rect: CGRect) -> Path {
        var path = Path()
        let t = WedgeTiles(wedge: wedge, in: rect)
        path.addArc(center: t.center, radius: t.innerRadius,
                    startAngle: t.start, endAngle: t.end, clockwise: false)
        path.addLine(to: t.outerEdgeStart)
        path.addArc(center: t.center, radius: t.outerRadius,
                    startAngle: t.end, endAngle: t.start, clockwise: true)
        path.closeSubpath()
        return path
    }
}
```

Custom view modifier as a transition:
```swift
struct ScaleFadeModifier: ViewModifier {
    var active: Bool
    func body(content: Content) -> some View {
        content
            .scaleEffect(active ? 0.01 : 1.0)
            .opacity(active ? 0 : 1)
    }
}

extension AnyTransition {
    static var scaleFade: AnyTransition {
        .modifier(active: ScaleFadeModifier(active: true),
                  identity: ScaleFadeModifier(active: false))
    }
}

// Apply:
WedgeView(wedge: wedge)
    .transition(.scaleFade)
```

`drawingGroup()` for Metal-accelerated compositing:
```swift
ZStack {
    ForEach(model.wedgeIDs, id: \.self) { id in
        WedgeView(wedge: model.wedges[id]!)
    }
}
.drawingGroup()
```

Custom alignment:
```swift
extension VerticalAlignment {
    private enum MidStarsAndTitle: AlignmentID {
        static func defaultValue(in d: ViewDimensions) -> CGFloat {
            d[.bottom]
        }
    }
    static let midStarsAndTitle = VerticalAlignment(MidStarsAndTitle.self)
}

HStack(alignment: .midStarsAndTitle) {
    StarRatingView()
        .alignmentGuide(.midStarsAndTitle) { d in d[VerticalAlignment.center] }
    TitleText()
        .alignmentGuide(.midStarsAndTitle) { d in d[VerticalAlignment.center] }
}
```

## Takeaways
- SwiftUI layout is a parent-proposes / child-decides negotiation; no view can be forced to a size — this eliminates over/under-constrained layout bugs entirely.
- Stack children are sized least-flexible first; use `layoutPriority` to ensure important content (like a title) gets space before decorative elements.
- Custom alignments project through nested stacks, enabling precise cross-hierarchy alignment without manual offset calculations.
- The `Shape` protocol unifies drawing and layout — shapes respond to layout proposals, receive modifiers, and animate via `animatableData` with no additional infrastructure.
- Apply `drawingGroup()` only to graphics-heavy subtrees; it routes all drawing through Metal but does not accelerate standard interactive controls.

---
_Source: WWDC19 Session 237 page (abstract and full transcript)._
