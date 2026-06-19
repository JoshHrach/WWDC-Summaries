# Create Great Enterprise Apps: A Chat with Box's Aaron Levie
**WWDC20 · Session 10204** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10204/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11

## Overview
This session is a conversation between Apple's Vice President of Cloud Services Mike Abbott and Box CEO and co-founder Aaron Levie about the evolving enterprise software landscape, with particular focus on how the shift to remote work in 2020 accelerated digital transformation across all industries. The discussion is strategic rather than technical, covering cloud architecture, mobile-first design principles, security considerations, and what enterprise developers should be building next.

Levie argues that the pandemic accelerated digital transformation by years — in some cases, by a decade — and that enterprise developers have an unprecedented opportunity to rebuild the underlying processes and experiences of large organizations from scratch for the 21st century. The central thesis is that consumer-grade experiences (fast, intuitive, device-native) should be the baseline expectation for enterprise software, not an aspirational goal.

The session also touches on how Box itself is building toward real-time collaboration with Apple Pencil support on iPad, zero-touch Mac deployment via Apple Business Manager and MDM, and microservices/continuous delivery practices.

## Key Topics

**Impact of Remote Work on Digital Transformation**
The shift to remote work compressed multi-year digital transformation roadmaps into quarters. Box's example: a performance initiative that would have involved 5–15 people on whiteboards became a 100-person Slack channel driving real-time architectural improvements.

**The Modern Digital Workplace**
Cloud-based content management (Box), real-time collaboration, video calling, and chat tools must work seamlessly across MacBooks, iPhones, and iPads. Zero-touch Mac deployment via Apple Business Manager lets a new employee be fully productive within minutes of first boot.

**Think Like a Consumer**
Enterprise software should match the quality and speed of consumer apps. Levie explicitly challenges the assumption that enterprise software can be slow or complicated "because it's enterprise software."

**Mobile-First and Digital-First Development**
Developers should design with mobile (iPhone/iPad) as the primary use case, not a secondary one. Box's iOS-first strategy means designing the best possible mobile experience before adapting to other platforms.

**Security by Design**
Security and data privacy must be baked into the application architecture from the start — not treated as a separate concern handled by the network or device. This is a requirement for serving regulated industries: hospitals, banks, government agencies, life sciences, K-12 education.

**Client vs. Cloud Logic**
A key architectural question: what belongs on the device versus in the cloud? On-device ML (Core ML), caching, and local processing can improve snappiness and privacy; cloud services handle shared state, collaboration, and heavy computation. The right split has shifted as devices (and HTML5 in browsers) have become far more capable.

**Microservices and Continuous Delivery**
Box deploys on a daily basis — not in quarterly releases — because the world changes too quickly. Microservices architecture allows teams to update individual capabilities without disrupting the whole application. This applies both to native app releases and to the cloud services they depend on.

**Apple Business Manager and MDM**
Zero-touch deployment (ordering a device that auto-enrolls and configures via MDM/Apple Business Manager) is a critical enabler of remote and hybrid work. Organizations like IBM use this to ship a Mac to any employee anywhere and have it fully set up without IT involvement.

## APIs & Frameworks

### Apple Platform Technologies Referenced
- **Core ML** — device-level machine learning for performance and privacy
- **MDM (Mobile Device Management)** — enterprise device enrollment and configuration
- **Apple Business Manager** — centralized device and app procurement and deployment
- **Apple Pencil (iPadOS)** — real-time annotation on documents (Box's upcoming feature)
- **iOS/iPadOS native APIs** — encryption and security capabilities built into device hardware
- **TestFlight** — beta distribution (mentioned in related sessions)

### Enterprise Architecture Patterns Discussed
- Microservices architecture for continuous, independent service deployments
- Cloud-first vs. device-first compute decisions
- Zero-touch MDM enrollment
- Continuous integration / continuous delivery (CI/CD)

## Code Highlights

No code samples were provided in this session. It is a strategic interview/discussion format.

## Takeaways
- COVID-19 forced enterprises to compress multi-year digital transformation plans into months; developers who understand this have an extraordinary opportunity right now.
- Consumer-grade quality (speed, simplicity, device-native design) must be the baseline for enterprise apps, not a premium feature — and Apple's device ecosystem is the primary driver of this expectation.
- Security must be architected into applications from day one, particularly for regulated industries; it cannot be delegated to the network or the device alone.
- The client/cloud compute split deserves deliberate design: on-device capabilities like Core ML have matured significantly and can replace server-side processing for performance and privacy-sensitive tasks.

---
_Source: WWDC20 Session 10204 page (abstract, chapter summaries, code samples, and resource links)._
