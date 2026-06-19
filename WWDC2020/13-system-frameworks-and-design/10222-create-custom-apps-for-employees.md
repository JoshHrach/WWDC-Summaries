# Create Custom Apps for Employees
**WWDC20 · Session 10222** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10222/)

_Platforms:_ iOS 14, iPadOS 14, watchOS 7

## Overview
This session focuses on the strategy and process for building enterprise apps — custom, employee-facing applications that help workers do their best work. Rather than a deep technical dive, the talk emphasizes identifying the right opportunities for mobile apps, engaging employees to drive the design process, and using Apple's development tools to iterate rapidly.

The session walks through the defining characteristics of enterprise apps: they are employee-facing, often distributed internally via MDM and Apple Business Manager rather than the public App Store, and exist across a wide spectrum — from simple company directories to complex role-specific workflows for pilots, field technicians, and retail staff. The common thread is that the best enterprise apps measurably improve an employee's job by reducing cognitive load and replacing cumbersome paper or legacy workflows.

Practical guidance covers where to find app opportunities (mobile workers, paper-heavy processes, hardware consolidation), how to conduct effective employee interviews, and how to use TestFlight and Xcode to deploy and iterate continuously based on real user feedback.

## Key Topics

**Characteristics of Enterprise Apps**
Enterprise apps are employee-facing and unique to their organization. They are typically distributed internally via MDM and Apple Business Manager, not the public App Store. They range from broad-use apps (company directories, room booking) to highly role-specific tools (pilot briefing apps, sales associate clienteling apps). Apps can work together in suites that share data via App Groups, Keychain, and Single Sign-On.

**Finding App Opportunities**
Key signals for strong app candidates:
- Mobile employees who work across locations (way-finding, offline support, location services for data accuracy)
- Paper-heavy processes ripe for digitization (aviation charts, insurance claims, inspection checklists)
- Employees carrying multiple devices that a single iPhone/iPad could consolidate (cameras, voice recorders, GPS units, measuring devices)
- Workflows that benefit from rich notifications with actionable quick responses

**Employee-Driven Design**
Effective enterprise app design starts with interviewing the actual workers — not their managers — who perform the job day-to-day. Best practices: ensure a safe, manager-free environment for honest feedback; interview a mix of new, mid-career, and veteran employees together; observe a full day in their life (not just 9–5); avoid assumptions and constantly ask "why"; maintain empathy throughout.

**Using Notification Content Extensions**
Custom notification UI (via Notification Content Extensions) enables employees to act on time-sensitive information without launching the app. Example: corporate pilots can view flight leg details and weather, then accept or decline assignments directly from a rich notification.

**Rapid Iteration with Apple Tools**
- Build and test quickly in Xcode using standard UIKit/SwiftUI views and controls to minimize custom UI learning curves.
- Use TestFlight for beta distribution and continuous feedback throughout the development lifecycle.
- Keep apps small, focused, and always evolving rather than building large monolithic apps.
- Distribute internally via MDM and Apple Business Manager.

**Hardware Capabilities Relevant to Enterprise**
- Location Services — real-time navigation, geofenced workflows, accurate data tagging
- Offline support — critical for remote field workers
- Wi-Fi, Bluetooth, and iBeacon — peer collaboration in environments without cellular
- Camera, microphone, GPS integration — device consolidation for field workers
- Apple Watch — wrist-based workflows for hands-free or quick-action use cases

## APIs & Frameworks

### Distribution & Management
- MDM (Mobile Device Management) — enterprise app distribution
- Apple Business Manager — centralized app and device management
- TestFlight — beta distribution and feedback collection

### Security & Identity
- Single Sign-On (SSO) — sign in once across a suite of enterprise apps
- App Groups — shared data container between related apps
- Keychain — shared credential storage across apps

### Notifications
- Notification Content Extensions — custom rich notification UI with interactive actions
- `UNNotificationContentExtension` — protocol for implementing interactive notification UI

### Location & Connectivity
- Location Services (Core Location) — way-finding, geofencing, location-tagged data
- Core Bluetooth — peer-to-peer collaboration without cellular
- iBeacon — indoor positioning for inspection and way-finding workflows

### Development Tools
- Xcode — primary IDE for building and testing
- SwiftUI / UIKit — standard views and controls recommended to minimize custom UI overhead

## Code Highlights

No code samples were provided in this session. The session is primarily strategic and process-oriented.

## Takeaways
- The most impactful enterprise apps are designed by talking directly to the employees who do the work — not managers or subject matter experts removed from the day-to-day reality.
- Look for app opportunities where employees are mobile, carry multiple devices, work with paper, or need quick actionable information; rich notifications and Apple Watch extensions can deliver value without even opening an app.
- Keep apps small, focused, and always evolving — distributing via TestFlight and collecting continuous feedback is more effective than large, infrequent releases.
- Standard UIKit/SwiftUI controls reduce training burden and development time; custom controls should be reserved for genuinely unique workflows.

---
_Source: WWDC20 Session 10222 page (abstract, chapter summaries, code samples, and resource links)._
