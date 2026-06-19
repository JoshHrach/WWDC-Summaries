# Prototyping: Fake It Till You Make It
**WWDC14 · Session 223** · [Watch](https://developer.apple.com/videos/play/wwdc2014/223/)

_Platforms:_ iOS 8, OS X Yosemite 10.10

## Overview
Presented by a small internal Apple prototyping team, this session teaches developers and designers how to validate ideas before writing production code by building progressively more realistic "fake apps." The core thesis is simple: make fake apps, show them to people, learn from feedback, and iterate until the experience feels great. This loop is demonstrated end-to-end by designing and prototyping a fictional artisanal toast-discovery app called "Toast Modern."

The session walks through three levels of fidelity: static picture prototypes (built in Keynote), animated transition prototypes (built with Keynote Magic Move), and interactive touch prototypes (built with Objective-C, Core Animation, and `UIImageView`-based layers in Xcode). Each level answers different questions and requires only minutes or hours rather than days of engineering effort.

The session is equally relevant to solo developers, team developers, designers, and non-technical managers, and explicitly encourages non-engineers to draw and animate without code using readily available tools.

## Key Topics

### Why Prototype
Going from idea directly to a finished app risks building the wrong thing. Inserting one or more prototype cycles between idea and code saves time and money by surfacing both problems (what does not work) and new ideas (what could work better). Two goals: test ideas and generate new ones.

### The Make–Show–Learn Loop
1. Ask: What needs to be more real? What can be faked? Where will people use it?
2. Build the smallest fake that makes the target experience testable.
3. Show it to the actual intended audience without coaching.
4. Ask: Do they understand it? Is it easy? How could it be better?
5. Learn: What is working? What is not? What new directions emerged?
6. Repeat until the experience feels great.

### Prototyping with Pictures (Keynote)
- Screenshot existing iOS apps and import into Keynote for Mac as layout references.
- Use Keynote's Shape tool to block UI regions, the Color Picker eye-dropper to match system colors, and Oval Mask to crop images.
- Repurpose Unicode Special Characters (stars, arrows) as placeholder icons.
- Use temporary filler text and reuse the same 2–3 photos for all screens.
- Export slides as PNGs and put them on device via Photo Stream, iCloud Drive, or email.
- Always zoom out to physical device size to check readability and tap-target size.

### Prototyping with Animation (Keynote)
- Keynote supports two animation types: **Build animations** (on individual shapes within a slide) and **Transition animations** (between slides, controlled via the Animate palette).
- **Magic Move** transition identifies same-named shapes on adjacent slides and interpolates position, size, and opacity — enabling keyframe-style animation without code.
- Stagger element positions and opacity on source/destination slides to create staggered list-item departure and arrival effects.
- Place shapes on both slides to prevent mid-animation fade-through-white artifacts.
- Duplicate slide A as slide C to automatically produce a reverse animation.
- Run the Keynote presentation on an iPhone to experience timing and feel on device.

### Prototyping with Interaction (Objective-C + Core Animation)
Three-step workflow:
1. **Put the picture on the device** — export Keynote slide as PNG at exact device pixel dimensions; load it into Xcode project; display it using a custom `Layer` class (subclass of `UIImageView`).
2. **Break up the picture** — create discrete PNG images for each interactive region (navigation bar, map, list) at larger-than-screen sizes to support panning/scrolling.
3. **Move pictures in response to touches** — implement `touchesMoved` to compute delta between `previousLocation` and `location` from `UITouch` and translate the image layer accordingly. Horizontal constraint for lists (y only); unconstrained for maps.

Supplementary techniques:
- Show a live `AVCaptureSession` camera preview beneath a fake camera UI overlay.
- Represent keyboard-entry sequences as a series of pre-rendered static images — no real `UITextField` or `UIKeyboard` needed.
- Hook a `touchesEnded` event on a navigation bar image to animate a camera panel in/out with a single `UIView.animate` call.

### What to Fake vs. What to Make Real
Each prototype cycle chooses one thing to make more real while faking everything else. Visual polish, real data, real navigation, and working keyboards are all fakeable. Only the specific interaction under test needs to be real.

## APIs & Frameworks

**Core Animation**
- `CALayer` — backing layer for all `UIView` subclasses; used directly for image compositing in prototypes
- `UIImageView` — base class for the prototype's custom `Layer` helper (which adds positional convenience methods and image loading)
- `UIView.animate(withDuration:animations:)` — used to animate panel appearances

**UIKit Touch Handling**
- `UITouch` — provides `location(in:)` and `previousLocation(in:)` for delta computation
- `touchesBegan(_:with:)`, `touchesMoved(_:with:)`, `touchesEnded(_:with:)` — UIResponder methods used to drive draggable image layers

**Keynote (macOS / iOS)**
- Magic Move transition (Keynote) — interpolates object position, size, opacity between slides for keyframe animation without code
- Build animations (Keynote) — per-object entrance/exit animations within a single slide
- Oval Mask (Keynote image masking) — used to crop images to circular shapes
- Special Characters panel (macOS) — source of Unicode symbols repurposed as placeholder icons

**Camera (for interactive prototype)**
- `AVFoundation` / `AVCaptureSession` — live camera preview shown beneath fake camera UI overlay (code not detailed; mentioned as a "realness" enhancement)

**Xcode**
- Xcode (any version supporting Objective-C + iOS 8 SDK) — used to build and run interactive prototypes on device

## Code Highlights

Custom `Layer` class (conceptual, Objective-C):
```objc
// Layer inherits from UIImageView for convenience positioning
// and image loading helpers. No production engineering needed.
@interface Layer : UIImageView
- (void)loadImage:(NSString *)name;
- (void)setPosition:(CGPoint)position;
@end
```

Drag-to-pan interaction (touch delta pattern):
```objc
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event {
    UITouch *touch = [touches anyObject];
    CGPoint current = [touch locationInView:self.superview];
    CGPoint previous = [touch previousLocationInView:self.superview];
    CGFloat dx = current.x - previous.x;
    CGFloat dy = current.y - previous.y;
    // For map: apply both axes
    mapLayer.center = CGPointMake(mapLayer.center.x + dx,
                                  mapLayer.center.y + dy);
    // For list: y-axis only
    listLayer.center = CGPointMake(listLayer.center.x,
                                   listLayer.center.y + dy);
}
```

Loading a full-screen prototype image:
```objc
Layer *screen = [[Layer alloc] initWithFrame:self.view.bounds];
Layer *picture = [[Layer alloc] init];
[picture loadImage:@"toast_list_screen"];
[screen addSubview:picture];
[self.view addSubview:screen];
```

## Takeaways

- Prototype fidelity should match the specific question being asked; never over-engineer a prototype — fake everything that isn't the thing under test.
- Keynote's Magic Move transition is a surprisingly powerful tool for exploring motion design and screen relationships without any code.
- For interactive touch prototypes, the minimal viable technique is to make `UIImageView` layers move in response to `UITouch` deltas — the resulting prototype can feel convincingly real in minutes.
- Showing prototypes to real users (not engineers or designers) and listening without defending is the highest-value activity in the entire prototyping cycle.

---
_Source: WWDC14 Session 223 page (abstract, chapter summaries, code samples, and resource links)._
