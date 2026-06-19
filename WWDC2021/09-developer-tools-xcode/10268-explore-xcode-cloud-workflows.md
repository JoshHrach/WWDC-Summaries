# Explore Xcode Cloud Workflows
**WWDC21 · Session 10268** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10268/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
Xcode Cloud workflows are the core configuration unit of Apple's cloud-hosted CI/CD system. A workflow defines when builds run (start conditions), how they run (environment: Xcode/macOS version, clean vs. incremental), what they do (actions: build, analyze, test, archive), and what happens afterward (post-actions: notifications, TestFlight deployment). Workflows are managed from both Xcode's Report Navigator and App Store Connect.

This session walks through creating a pull-request workflow end to end, explains all configuration sections, and presents three recommended workflow patterns: pull request, release/distribution, and overnight testing.

## Key Topics

**Start Conditions**
Four condition types control when a workflow fires:
- _Every Change to a Branch_ — builds the branch directly, ignoring PR state
- _Every Change to a Pull Request_ — builds the merge of source + target branches **[NEW]**; can filter to specific files/folders; supports auto-cancel of in-progress builds
- _Every Change to a Tag_ — fires on new tag creation
- _On a Schedule_ — recurring schedule (e.g., weeknights at 1 a.m.)

**Environment**
Selects Xcode and macOS versions (specific, Latest Release, or Latest Beta). "Clean" forces a full rebuild (required for external TestFlight / App Store); omitting it enables incremental builds for faster CI. Environment variables (including secrets) are available to custom build scripts.

**Actions**
- **Build** — basic compile check for schemes/configurations not covered by other actions
- **Analyze** — static analysis; can be marked Required to Pass or informational
- **Test** — runs tests on selected simulators; supports Use Scheme Settings or a named test plan; simulators can be pinned to specific OS versions
- **Archive** — produces a signed artifact; deployment mode: None, TestFlight Internal Testing Only, or TestFlight and App Store (required for external testing/App Store). Handles code signing automatically via Cloud Signing.

**Post-Actions**
- **Notify** — Slack channel integration and/or email addresses; configurable for all successes/fixes/failures/breaks
- **TestFlight Internal Testing** — auto-deploys to internal groups; works with PR and incremental builds
- **TestFlight External Testing** — requires single branch start condition, Clean build, and TestFlight and App Store deployment setting; supports external groups and individual testers

**Recommended Workflow Patterns**
1. _Pull Request workflow_ — auto-triggers on PR, runs analyze + test + archive, deploys to QA internal group, notifies CI channel
2. _Release workflow_ — triggers on release branch, Clean build, pinned Xcode version, comprehensive tests on multiple simulators, deploys to external TestFlight group
3. _Overnight Testing workflow_ — scheduled nightly, latest Xcode, multiple test plans + simulators all required to pass, notifies QA on failures only

## APIs & Frameworks

- **Xcode Cloud** **[NEW]** — cloud-hosted CI/CD service
- Xcode Report Navigator → Cloud tab **[NEW]** — view workflows and builds from teammates
- Workflow configuration (Xcode UI: Product > Manage Workflows) **[NEW]**
- Workflow components:
  - `General` — name, edit restriction, primary repository, project/workspace
  - `Start Conditions` — condition type, branch/tag/PR filters, file/folder filters, auto-cancel
  - `Environment` — Xcode version, macOS version, Clean flag, environment variables (secrets supported) **[NEW]**
  - `Actions` — Build, Analyze, Test, Archive (with deployment preparation setting)
  - `Post-Actions` — Notify (Slack + email), TestFlight Internal Testing, TestFlight External Testing
- **Xcode Cloud REST API** — programmatic workflow management **[NEW]**
- **Custom build scripts** — run on CI machines, access environment variables **[NEW]**
- **Cloud Signing** — automatic provisioning profiles and code-signing identities **[NEW]**
- **TestFlight** — internal and external group management via App Store Connect
- **App Store Connect** — alternative UI for workflow management **[NEW]**
- Start condition type: `Every Change to a Pull Request` **[NEW]** — builds merged source + target
- Start condition type: `Every Change to a Branch`
- Start condition type: `Every Change to a Tag`
- Start condition type: `On a Schedule` — with weekly frequency/day/time selection
- Action: `Test` — `Recommended Simulators` option covers multiple screen sizes automatically
- Action: `Archive` — deployment preparation: `TestFlight Internal Testing Only` / `TestFlight and App Store`
- Post-action: `TestFlight Internal Testing` / `TestFlight External Testing`
- Notification events: `Build Success` (All / Fixes / Don't Notify), `Build Failure` (All / Breaks / Don't Notify)

## Code Highlights

No Swift/Objective-C code samples — this is a tooling/workflow configuration session. Configuration is done via Xcode's workflow editor UI or App Store Connect. Custom scripts can be added as shell scripts in the repository at conventional paths recognized by Xcode Cloud.

## Takeaways

- Pull-request start conditions build the actual merge of source and target branches, giving teams confidence that changes integrate cleanly before merging.
- Clean builds are required for external TestFlight distribution and App Store submission; for PR and internal workflows, incremental builds significantly reduce CI time.
- Post-actions (Slack notifications, TestFlight deployment) are configured per workflow, enabling fully automated delivery pipelines from commit to tester.
- Xcode Cloud handles code signing automatically — no manual certificate/profile management needed for CI builds.

---
_Source: WWDC21 Session 10268 page (abstract, chapter summaries, code samples, and resource links)._
