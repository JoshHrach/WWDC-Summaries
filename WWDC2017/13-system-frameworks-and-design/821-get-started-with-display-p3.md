# Get Started with Display P3
**WWDC17 · Session 821** · [Watch](https://developer.apple.com/videos/play/wwdc2017/821/)

_Platforms:_ iOS 11, macOS High Sierra 10.13, tvOS 11, watchOS 4

## Overview
Wide color displays allow apps to present richer, more vibrant, and lifelike colors than the longstanding sRGB standard. Display P3 is a color space within the RGB color model that covers 25 percent more colors than sRGB, making it better suited for photography, scenic imagery, textile/fashion apps, and any UI that relies on saturated, vivid hues. Apple introduced wide-gamut display support with the late-2015 iMac Retina 5K and has since expanded it to the iPhone 7, iPad Pro, and MacBook Pro with Touch Bar.

Color management is built into both macOS and iOS and ensures that colors are accurately represented across devices and color spaces. Properly tagging assets with the correct color profile is essential so that the OS can perform accurate color matching without unexpected hue shifts. Designers who skip tagging risk having colors rendered incorrectly on wide-gamut hardware.

The session focuses on practical Photoshop-based workflow techniques for creating Display P3 assets: creating documents with the Display P3 profile, raising bit depth to 16 bits per channel to eliminate gradient banding, and exporting via Save As PNG (the only path that preserves both an embedded profile and full color depth).

## Key Topics
- **Color management fundamentals** — how macOS and iOS translate colors across devices and why tagging assets matters
- **What is Display P3** — 25 % larger gamut than sRGB; informally called "wide gamut"; introduced on late-2015 iMac Retina
- **Compatible Apple devices** — iMac Retina (late 2015+), iPhone 7, iPad Pro, MacBook Pro with Touch Bar, LG 5K display
- **16-bit color depth** — reduces gradient banding within the wider gamut; compared against sRGB 8-bit banding example
- **Photoshop workflow** — setting the color profile to Display P3 on document creation, changing bit depth to 16 bpc, using Convert to Profile (not Assign to Profile) for existing assets
- **Export pitfalls** — only Save As PNG embeds the color profile and preserves bit depth; Photoshop Generator omits color profiles entirely
- **Raw file pipeline** — convert to Display P3 directly from raw files; converting from sRGB loses irretrievable color data
- **When to use Display P3** — scenic photography, nature/travel imagery, textile/fashion apps, vibrant UI accents (VU meters, rev counters); not recommended for calm/de-saturated UI schemes

## APIs & Frameworks
Display P3 adoption is primarily a design/asset-pipeline concern rather than a code-heavy API surface, but the following are relevant:

- **Display P3 color profile** — ICC color profile applied to images and documents **[NEW adoption]**
- **Color management (macOS / iOS)** — built-in OS-level color management pipeline that maps tagged assets to the display's native gamut
- **`UIColor` / `NSColor`** — support for extended-range (wide-gamut) color values on Display P3-capable hardware
- **Asset Catalogs (Xcode)** — store Display P3 variants of images and app icons alongside sRGB variants
- **PNG with embedded ICC profile** — the required export format to preserve Display P3 tagging and 16-bit depth
- **Photoshop "Convert to Profile"** — performs color-match conversion preserving visual appearance when migrating sRGB assets to Display P3
- **Photoshop "Save As" (PNG)** — only export path that retains both embedded color profile and 16-bit-per-channel depth

## Code Highlights
This session is design-workflow focused; no code samples were presented. The key "code" decisions are in the asset pipeline:

- Create Photoshop document → set **Color Profile: Display P3**, **Bit Depth: 16 bits/channel**
- For existing sRGB assets: **Edit → Convert to Profile → Display P3** (do NOT use Assign to Profile)
- Export via **File → Save As → PNG** (not Generator, not Export As) to embed the ICC profile

## Takeaways
- Display P3 provides a 25 % larger color gamut than sRGB and is already supported on tens of millions of Apple devices; adoption is straightforward for most visual apps.
- Always tag assets with the Display P3 color profile and export via Save As PNG to ensure color management works correctly end-to-end.
- Use 16 bits per channel in Photoshop to minimize gradient banding within the wider gamut.
- Display P3 is most impactful for scenic photography, travel, fashion/textile apps, and saturated UI accents; it adds little value for de-saturated, calm UI designs.

---
_Source: WWDC17 Session 821 page (abstract, chapter summaries, code samples, and resource links)._
