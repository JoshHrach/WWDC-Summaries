# Create Practical Workflows in Xcode Cloud
**WWDC23 · Session 10278** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10278/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10, tvOS 17

## Overview
This session teaches Xcode Cloud workflow design through three real-world case studies: a solo developer, a medium-sized cross-functional team, and a large team migrating from an in-house CI system. Rather than covering APIs, it shows how to compose the "what, where, and when" elements of Xcode Cloud workflows to automate CI/CD pipelines of varying complexity.

Key concepts include: multi-platform archive actions, PR-triggered test workflows, tag-based release workflows, custom pre/post-clone/build scripts for tasks like swapping app icons, the "Not Required To Pass" test action option for safely onboarding unreliable test suites, workflow duplication, and integration with external systems via webhooks and the Xcode Cloud public REST API.

## Key Topics

### Workflow Fundamentals
An Xcode Cloud workflow answers three questions:
- **What**: Actions — Build, Test, Archive, Analyze; and Post-actions — TestFlight Internal/External Testing, Notify (email/Slack).
- **Where**: Environment — Xcode version, macOS version, environment variables, custom scripts.
- **When**: Start Conditions — Branch Changes, Tag Changes, Pull Request Changes, Schedule.

### Case Study 1: Solo Developer
Single "CI Workflow" covering both iOS and macOS:
- Start condition: Branch Changes on Any Branch.
- Two Archive actions (iOS and macOS), each with "TestFlight and App Store" deployment preparation.
- Two TestFlight External Testing post-actions distributing to a "Friends and Family" group.
- Cocoapods handled via a post-clone custom script (`ci_post_clone.sh`) that runs `pod install`.

### Case Study 2: Medium-Sized Team (Three Workflows)

**Pull Request Workflow**:
- Start: Pull Request Changes targeting the `beta` branch only.
- Action: Test on four destination devices (small iPhone, large iPhone, small iPad, large iPad).

**Beta Release Workflow**:
- Start: Branch Changes on `beta`.
- Action: Archive with "TestFlight Internal Testing" deployment preparation.
- Post-action: TestFlight Internal Testing to "QA Team" group.
- Safety net: same four-device test action also included.
- Custom pre-build script swaps app icon for beta builds using `$CI_XCODEBUILD_ACTION` and `$CI_WORKFLOW` environment variables.

**Release Workflow** (duplicated from Beta):
- Start: Tag Changes where tag begins with `release/`.
- Archive with "TestFlight and App Store" deployment preparation.
- Post-actions: TestFlight Internal Testing to "Executive Stakeholders"; TestFlight External Testing to "Early Adopters".
- Notify post-action posting build results to a Slack "Releases Feed" channel.

### Case Study 3: Large Team Migration (Three Milestones)

**Milestone 1 — Release Workflow**:
Start with an archive workflow for App Store builds. Use Xcode Cloud's built-in cloud code signing (no manual cert/profile management). This reveals any dependency/config issues without disrupting the team.

**Milestone 2 — Reliable Tests**:
Run all tests with "Not Required To Pass" marking at first, so CI failures don't block PRs while the team evaluates reliability. Move reliable tests into a dedicated Test Plan. Add a second, required test action running only the Reliable Tests plan. Gradually migrate remaining tests as they are fixed.

**Milestone 3 — Complete Remaining Workflows**:
Build out beta, release, and other workflows. Integrate with external tools:
- **Webhooks**: Xcode Cloud sends HTTP POST to your server after each build; use to create QA tickets, update dashboards, etc.
- **Xcode Cloud Public API (App Store Connect API)**: Fetch build data to populate status pages or dashboards.

### Custom Build Scripts
Three hook points: `ci_post_clone.sh`, `ci_pre_xcodebuild.sh`, `ci_post_xcodebuild.sh`. Scripts use Xcode Cloud environment variables (`$CI_WORKFLOW`, `$CI_XCODEBUILD_ACTION`, `$CI_BRANCH`, `$CI_TAG`, etc.) to conditionally modify behavior. Useful for dependency managers (Cocoapods, Carthage), app icon swapping, metadata updates, and more.

### Not Required To Pass (Test Action Option)
Setting a Test action's "Requirement" to "Not Required To Pass" lets the overall build succeed even if tests fail. The green checkmark still appears in Xcode Cloud and in the source code management commit status. This is the recommended mechanism for safely onboarding legacy or unreliable test suites.

### Workflow Duplication
Right-click any workflow in the workflow editor → Duplicate to clone the entire configuration as a starting point for a new workflow. Reduces repetitive setup.

## APIs & Frameworks

**Xcode Cloud (Workflow Configuration)**
- Start Conditions: Branch Changes, Tag Changes, Pull Request Changes, Schedule
- Branch/tag filtering: Any Branch, Custom Branches, tags beginning with pattern
- Actions: Build, Test, Archive, Analyze
- Post-Actions: TestFlight Internal Testing, TestFlight External Testing, Notify (email, Slack)
- Test action "Requirement: Not Required To Pass" **[highlighted feature]**
- Archive "Deployment Preparation": TestFlight and App Store, TestFlight Internal Testing

**Custom Scripts**
- `ci_post_clone.sh` — runs after source clone (for dependency managers)
- `ci_pre_xcodebuild.sh` — runs before build/archive/test
- `ci_post_xcodebuild.sh` — runs after build/archive/test
- Environment variables: `$CI_WORKFLOW`, `$CI_XCODEBUILD_ACTION`, `$CI_BRANCH`, `$CI_TAG`, `$CI_BUILD_NUMBER`, and many more (full list in documentation)

**External Integration**
- Xcode Cloud Webhooks — HTTP POST on build completion with build/workflow metadata
- Xcode Cloud Public API (App Store Connect API) — REST API for builds, workflows, artifacts; for dashboards/status pages

## Code Highlights

Pre-build script for swapping beta app icon:
```sh
#!/bin/sh
# ci_pre_xcodebuild.sh
if [[ "$CI_XCODEBUILD_ACTION" == "archive" && "$CI_WORKFLOW" == "Beta" ]]; then
    echo "Replacing app icon with beta icon"
    mv BetaAppIcon.appiconset ../App/Assets.xcassets/AppIcon.appiconset
fi
```

## Takeaways
- Three well-scoped workflows (PR/beta/release) cover the full CI/CD lifecycle for most teams; start simple and layer complexity.
- "Not Required To Pass" on test actions is the practical solution for migrating legacy test suites to Xcode Cloud without blocking the team.
- Custom scripts at three hook points (`post-clone`, `pre-xcodebuild`, `post-xcodebuild`) and the full set of environment variables make Xcode Cloud extensible for nearly any workflow need.
- Webhooks and the App Store Connect API let teams integrate Xcode Cloud with external tools (project management, dashboards, Slack) without custom build machines.

---
_Source: WWDC23 Session 10278 page (abstract, chapter summaries, code samples, and resource links)._
