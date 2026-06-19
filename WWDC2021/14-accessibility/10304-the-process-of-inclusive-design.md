# The Process of Inclusive Design
**WWDC21 · Session 10304** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10304/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
Where the companion session "The practice of inclusive design" focuses on specific design techniques for content, this session focuses on organizational and process changes that enable teams to build more inclusive products. Presented by Caroline Cranfill (Apple Design), Cynthia Bennett (AI and Machine Learning research), and Sabrine Rekik (Camera & Photos engineering), the session argues that inclusion must be embedded into every phase of development — not added at the end.

The session challenges three common misconceptions: that inclusion is too hard, that it limits creativity, and that a diverse team automatically delivers inclusive results. It reframes inclusion as an ongoing journey requiring empowered diverse voices, protected reflection time throughout the development process, and genuine engagement with people at the intersections of multiple diversity axes.

A detailed case study from the Photos Memories feature illustrates the full loop: gathering diverse feedback, running brainstorming sessions with international employees, conducting one-on-one interviews to understand emotional impact, iterating on designs validated with the original feedback providers, and continuing to improve after launch.

## Key Topics

### Reframing Common Misconceptions
- Inclusion is a journey, not a one-time task — iterative, requiring patience and persistence
- Inclusive design stimulates creativity by surfacing constraints that sharpen problem-solving
- Diverse teams must be empowered, not just assembled — different perspectives must be invited and used in decision-making

### Diversity Axes Framework
- Key axes: class, culture, ethnicity, language, education, beliefs, race, gender, sexual orientation, age, abilities/disabilities, handedness, body measurements, environmental conditions (location, connectivity, device access)
- Intersectionality: unique experiences emerge at intersections of multiple axes; designing for one group can inadvertently harm another if intersections are not considered

### Inclusion in the Design Process (Phase by Phase)
**Ideation:**
- Answer: Why are you making this? Who is it for? Which diversity axes have been considered?
- Actively listen to perspectives beyond your team's own lived experiences

**Design:**
- Consider extremes, not just the average user — identify unconsidered intersections
- Evaluate emotional, social, and physical impact; are vulnerable populations disadvantaged?
- Account for context: culture, environment, connectivity, household composition

**Development:**
- Include diverse voices in prototype evaluation and decision-making
- Plan for accessibility and internationalization from the start, not the end
- Standard UIKit, AppKit, and SwiftUI controls are accessible by default; custom controls require additional work

**Testing:**
- Seek a diverse group of testers reflecting real-world users
- Test with assistive technologies: VoiceOver, maximum text size, keyboard navigation
- Make inclusive testing a KPI alongside performance and crash metrics

**Release:**
- Make hard calls to cut features rather than release poor experiences for certain groups
- Document known limitations with a plan and timeline to address them

**Post-Launch:**
- Gather and prioritize feedback from people with experiences different from the development team
- Use brainstorming sessions (smaller, international groups) and individual interviews (for sensitive topics)
- Validate redesigns with the same people who raised the original concerns

### Case Study: VoiceOver Recognition Image Descriptions (Cynthia Bennett)
- Machine learning model generates automatic image descriptions for people who are blind or have low vision
- Issue discovered: ML models typically described only two genders (female/male), excluding non-binary and transgender people
- Intersectionality addressed: blind/low-vision users also span the gender diversity spectrum
- Solution: partnered with external organizations at the intersection of blindness and LGBTQ+ communities; developed gender-neutral image descriptions
- iOS 15 Markup: users can write their own photo descriptions that travel with shared images

### Case Study: Photos Memories Feature (Sabrine Rekik)
- Feedback collection from App Store reviews and direct user outreach
- Goal 1 — Celebrate diversity: new Memory themes for hobbies (martial arts, skateboarding, soccer) and international holidays (Diwali, Lunar New Year, Eid al-Fitr, Hanukkah) via collaboration with cultural experts
- Goal 2 — Sensitivity to difficult experiences: one-on-one interviews revealed emotional impact of unwanted people appearing in Memories
- Design evolution: added "Feature Less" → "Never Feature" options, available from Featured photo, People albums, and single asset views
- Icon iteration: changed thumbs down glyph (implying dislike of a person) to a neutral glyph

## APIs & Frameworks

- `UIKit` standard controls (accessible by default via `UIAccessibility`)
- `AppKit` standard controls (accessible by default)
- `SwiftUI` standard views (accessible by default)
- `UIAccessibility` protocol
- VoiceOver Recognition (on-device ML for image descriptions) **[NEW in iOS 15]**
- Photos Markup description feature **[NEW in iOS 15]**
- `AVFoundation` (audio/captioning)
- Human Interface Guidelines: Inclusion (referenced resource)

## Code Highlights

No code samples included in this session. The session is process- and organization-focused.

Key accessibility API patterns referenced:
- Custom views must implement `UIAccessibility` methods manually; standard UIKit/AppKit/SwiftUI views do so automatically
- Internationalization and localization (`NSLocalizedString`, `Locale`, `Calendar`) must be planned from the start of development

## Takeaways

- Inclusion is not a phase at the end of development — it requires dedicated time for reflection, validation, and iteration at every stage of the process.
- Diversity axes and their intersections are a practical framework for asking "who are we excluding?" during ideation and design reviews.
- Standard Apple frameworks (UIKit, AppKit, SwiftUI) provide accessibility for free; custom controls require explicit accessibility implementation and should be planned from the beginning.
- Post-launch is not the end: user feedback, one-on-one interviews, and collaborative iteration with affected communities are how inclusive experiences continue to improve.

---
_Source: WWDC21 Session 10304 page (abstract, chapter summaries, code samples, and resource links)._
