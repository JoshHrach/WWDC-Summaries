# App Icon Design
**WWDC17 · Session 822** · [Watch](https://developer.apple.com/videos/play/wwdc2017/822/)

_Platforms:_ iOS 11, macOS High Sierra 10.13, watchOS 4, tvOS 11

## Overview
This session walks through the foundational design principles behind effective app icons, drawing on Apple's own history from classic Mac OS icons through modern iOS and macOS. The presenter argues that app icons are symbols — part of a long human tradition of using visual marks to communicate ideas — and that the best icons apply a consistent set of principles regardless of platform or era.

Four core principles are introduced and illustrated with Apple's own apps: metaphor (using a recognizable real-world object or graphic mark), simplicity (keeping the design clean, uncluttered, and immediately readable at any size), connection (creating an emotional bond between the user and the app through the icon), and lineage (evolving the icon deliberately over time so users maintain a sense of familiarity with the brand).

The session also covers practical testing techniques — examining icons at small sizes, inside folders, squinting to judge contrast and recognizability, and comparing against competitor icons in the same category — along with design-process advice: sketch on paper first, iterate, save early directions, and be patient.

## Key Topics

- **Historical context of app iconography** — classic Mac OS icons (paint bucket, floppy disk, bomb) as examples of metaphor, line weight, implied movement, and black/white contrast; principles still valid today.
- **Four design principles** — metaphor, simplicity, emotional connection, and lineage; illustrated through the evolution of the Keynote icon (podium) from 2003 through modern iOS/macOS versions.
- **Cross-platform icon families** — iWork (Keynote/Pages/Numbers) and iLife (GarageBand/iMovie) as examples of shared color palettes and glyph language tying iOS and macOS icons together; importance of visual consistency signaling a unified experience.
- **iOS icon grid** — using the system grid so common elements (circles, shapes) are the same size across apps; example with Music Memos vs. Safari.
- **News icon case study** — iterating through three icon concepts to find the design with the clearest visible newspaper metaphor, best use of background color, and highest contrast at small size.
- **Uniqueness within a category** — analyzing icons from Quick, Instagram, and Periscope to see how distinct colors, shapes, and compositions make each stand out.
- **Practical testing** — viewing icons on the Home screen, inside folders (small size), in Settings; squinting test for contrast; comparing with neighboring icons.
- **Design process** — sketching on paper, experimenting broadly, testing early and often, optimizing assets for multiple device sizes, patience with the iterative process.

## APIs & Frameworks

This is a design-principles session with no code content. Tools and resources referenced:

- **iOS icon grid** — Apple's system icon geometry guide for aligning icon elements to shared proportions
- **Human Interface Guidelines** — referenced for icon design best practices (`developer.apple.com/design/`)
- **Xcode Asset Catalogs** — implied context for delivering icons at multiple resolutions for different device sizes
- **Apple Design Resources** — linked via the Apple Design Site (`developer.apple.com/design/`)

No specific APIs are introduced or discussed.

## Code Highlights

No code samples. This session is entirely a visual design lecture.

Key icon design checks to perform before shipping:

1. Place the icon on a real Home screen at actual device size.
2. Put it in a folder alongside other icons and squint — does it still read?
3. Compare it against the top apps in your App Store category.
4. Test it inside the Settings app (if the app appears there).
5. Verify assets are exported at all required resolutions for each target platform.

## Takeaways

- Every great app icon needs a clear metaphor — a simple, recognizable object or mark that communicates what the app does without relying on the label.
- Simplicity and high contrast are non-negotiable: a design that looks great at 1024×1024 must still be instantly readable at 29×29 inside a folder.
- Treat the icon as a long-term brand investment: refine deliberately over time rather than redesigning with each release, so users maintain familiarity.
- Test, test, test — the Home screen, folder view, and squint test reveal problems that look invisible in a design tool at full size.

---
_Source: WWDC17 Session 822 page (abstract, transcript, and resource links)._
