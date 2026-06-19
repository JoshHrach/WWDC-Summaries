# Developer spotlight: Accessibility
**WWDC21 · Session 10318** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10318/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This short documentary-style session features developers who are blind or deaf sharing first-hand perspectives on building accessible apps. Rather than a technical tutorial, it focuses on the mindset shift required to design and test for accessibility from the beginning, illustrated through real products: Seeing AI (Microsoft), Cardzilla, and Pro Tools (Avid).

The central message is that accessibility is not a niche feature or a large burden—adding labels to graphical buttons can be done in a day and unlock an app for users who would otherwise be entirely excluded. The session makes the case that diverse teams, including developers with disabilities, produce better software for everyone.

## Key Topics

### Accessibility as Universal Design
- Accessibility benefits a billion people worldwide with some form of disability; it is not a niche audience.
- The goal is one universally accessible version of an app, not a separate "accessible version."
- Good accessibility design frequently improves usability for all users, not just those with disabilities.

### Starting Small Has Big Impact
- The most common misconception: accessibility is a lot of work. In practice, adding accessibility labels to graphical buttons is often a one-day fix.
- VoiceOver accessibility labels on buttons, accessible caption bars, and large text displays are examples of small investments with high impact.
- Long-term accessibility is a sustained design investment, not a one-time feature.

### Developer Perspectives Featured
- **Avid / Pro Tools**: A developer who lost their sight in 2009 and now uses a screen reader daily for coding; advocates for one universally accessible tool.
- **Twitter**: A deaf developer who relies entirely on visual feedback and uses the VoiceOver caption bar (added by Apple) to test VoiceOver—demonstrating that accessibility tools themselves must be accessible.
- **Microsoft / Seeing AI**: The app describes the world to blind users—reading text, identifying objects, describing people. Innovations driven by lived experience include audio "hot or cold" guidance for barcode scanning (faster beeps as the barcode comes closer).
- **Cardzilla**: A deaf developer built this app because Notes text was too small for taxi drivers to read; large-text communication solved a daily friction point.

### Diversity as a Design Engine
- Having team members with disabilities surfaces problems that sighted/hearing developers never encounter.
- Testing in real-world conditions with real-world constraints reveals issues that standard QA misses.
- User stories from accessibility users are often the most emotionally powerful validation of an app's impact.

### VoiceOver Caption Bar
- VoiceOver on iOS provides a caption bar that displays what VoiceOver is speaking as text.
- This allows deaf developers and testers to verify VoiceOver behavior without hearing the audio—an example of Apple making its own accessibility tools accessible.

## APIs & Frameworks

This session is narrative/inspirational, not a technical API walkthrough. The following APIs are mentioned or implied through the developer stories:

**Accessibility framework**
- `accessibilityLabel` — text description of a UI element read by VoiceOver **[existing]**
- VoiceOver — screen reader used daily by blind developers **[existing]**
- VoiceOver caption bar — displays VoiceOver speech as text on-screen, enabling deaf testers **[existing]**

**UIKit / AppKit**
- `UIAccessibility` — protocol properties for labeling, traits, hints **[existing]**
- `accessibilityTraits` — describes element behavior (button, image, header, etc.) **[existing]**

**Vision / Seeing AI (mentioned)**
- Text recognition, object identification, person recognition — referenced as capabilities of Seeing AI (Microsoft) **[existing platform technology]**

**Speech / Dictation**
- Speech-to-text for communication (Cardzilla's dictation feature) — referenced as enabling cross-disability communication **[existing platform technology]**

## Code Highlights

No code samples were presented; this is an inspirational documentary session.

The practical accessibility starting point emphasized:
- Assign meaningful `accessibilityLabel` values to all graphical buttons (images, icons).
- Test with VoiceOver enabled; use the caption bar if unable to hear audio output.
- Engage beta testers with disabilities early and iteratively.

## Takeaways
- Accessibility is not a heavy burden: labeling graphical buttons can be done in a day and immediately opens an app to blind users.
- Build one universally accessible product, not a separate "accessible" variant.
- Team diversity—including developers who are blind, deaf, or have other disabilities—is the most reliable source of accessible design insights.
- Platform accessibility tools (VoiceOver, caption bar, large text) must themselves be accessible; Apple's decision to add the VoiceOver caption bar enabled deaf developers to test VoiceOver behavior visually.

---
_Source: WWDC21 Session 10318 page (abstract, full transcript)._
