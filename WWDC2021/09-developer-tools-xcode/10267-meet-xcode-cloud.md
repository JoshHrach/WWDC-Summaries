# Meet Xcode Cloud
**WWDC21 · Session 10267** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10267/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
Xcode Cloud is Apple's continuous integration and continuous delivery (CI/CD) service built directly into Xcode and App Store Connect. It automates building, testing, and distributing apps and frameworks for all Apple platforms, helping teams catch issues early and iterate quickly on both code and feedback.

The service runs builds on Apple-managed cloud infrastructure that provides code signing, access to multiple OS versions, and multiple Xcode releases. Build environments are ephemeral — workloads are fully isolated, source code is never stored, and build data is encrypted at rest in a dedicated CloudKit database.

Xcode Cloud connects the entire development pipeline: write code in Xcode, review pull requests, distribute via TestFlight, and collect feedback — all within a single integrated workflow. Teams can configure personalized Slack or email notifications, and every team member can monitor the same shared workflows from Xcode or App Store Connect.

## Key Topics
- **Introduction to Continuous Integration** — How CI helps teams integrate changes regularly, catch issues early, and maintain a stable release candidate.
- **Xcode Cloud Overview** — Tour of the feature set: workflows, builds, build group overview, build reports, App Store Connect web interface, and privacy model.
- **Onboarding a Project** — Step-by-step setup using the "Create Workflow" option in the Product menu; automatic source repository discovery and authorization with GitHub (or other providers).
- **Build Reports** — Viewing action status, logs, artifacts, and jumping directly to problematic code from within Xcode.
- **Workflow Editing** — Adding start conditions (push to branch, additional branches), adding Test actions, selecting simulator destinations, and saving updated workflows.
- **Manual Build Triggering** — Starting a build on demand for a specific branch without pushing a code change.
- **Privacy and Security** — Temporary build environments, isolated workloads, no source code storage, encrypted build data, user-controlled data deletion.

## APIs & Frameworks
- **Xcode Cloud** **[NEW]** — Top-level CI/CD service integrated into Xcode and App Store Connect.
  - Workflow configuration (start condition, environment, actions, post-actions)
  - Archive action **[NEW]**
  - Test action **[NEW]**
  - Build group overview **[NEW]**
  - Build report (logs, artifacts, issue jump) **[NEW]**
  - Manual build trigger **[NEW]**
  - Notification settings (Slack, email) **[NEW]**
- **App Store Connect** — Web-based Xcode Cloud management: start/view builds, manage workflows, download artifacts, manage notifications.
- **TestFlight** — Direct integration; Xcode Cloud distributes builds to TestFlight as a post-action.
- **CloudKit** — Used internally to store encrypted build data in a dedicated database.
- **Conditional compilation (`#if !os(macOS)`)** — Illustrated in the code fix during the demo.

## Code Highlights
Fixing a platform-specific import that Xcode Cloud's first build caught:

```swift
import SwiftUI
#if !os(macOS)
import UIKit
#endif
```

## Takeaways
- Xcode Cloud requires minimal setup — a few clicks in the Product menu connects a project and kicks off its first build.
- Workflows are shared across the entire team; any member with the project open in Xcode sees the same workflows and build history.
- Build environments are fully ephemeral and source code is never persisted, making privacy a first-class concern.
- Xcode Cloud surfaces build issues directly in the code editor, enabling rapid triage without leaving Xcode.

---
_Source: WWDC21 Session 10267 page (abstract, chapter summaries, code samples, and resource links)._
