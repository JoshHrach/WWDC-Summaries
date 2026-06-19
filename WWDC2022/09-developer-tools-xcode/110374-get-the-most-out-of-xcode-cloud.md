# Get the most out of Xcode Cloud
**WWDC22 · Session 110374** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110374/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9 (tool-agnostic CI/CD service)

## Overview
Xcode Cloud is Apple's CI/CD service built into Xcode and App Store Connect. It automatically builds, tests, analyzes, and archives apps in the cloud and distributes them to testers. This session demonstrates how to read and act on the new Xcode Cloud Usage Dashboard in App Store Connect, then walks through concrete workflow optimizations — refined start conditions, targeted test destinations, and the new `ci skip` commit tag — that reduce both build duration and compute usage.

The session uses a "Food Truck" sample project as a running example, starting from a 14-minute build consuming 32 minutes of compute and showing how best-practice changes reduce that usage, freeing up headroom to add a new watchOS companion target without hitting compute limits.

## Key Topics

**Build duration vs. compute usage** — Xcode Cloud runs actions (Analyze, Archive, Build, Test) in parallel. Build duration equals the longest-running action; compute usage is the sum of all individual action durations. Optimizing parallel actions reduces usage even when duration stays the same.

**Xcode Cloud Usage Dashboard (new in App Store Connect)** — A new App Store Connect view showing: current-cycle compute minutes used/remaining, percentage used, usage trends (builds created and total minutes) over 30-day or custom periods, per-product breakdowns, and per-workflow breakdowns. Use it to identify which workflows consume the most compute and track the effect of optimizations over time.

**Start Conditions best practices** — Replace scheduled triggers with Branch Changes triggers to avoid building commits that have already been built. Restrict to specific branch prefixes (e.g., `release*`) rather than "Any Branch." Exclude non-source folders (e.g., a `docs/` folder) using the "Files and Folders" custom condition to prevent unnecessary builds.

**Test Destination optimization** — Running tests on many similar simulator destinations adds compute without adding signal. Xcode Cloud provides a "Recommended iPhones" alias — a curated cross-section of screen sizes — that gives meaningful coverage with fewer destinations.

**`ci skip` commit tag (new)** — Append `ci skip` to any commit message to tell Xcode Cloud to ignore that push event entirely. Useful for documentation-only commits, formatting changes, or other non-functional changes that do not need a CI run.

**Custom scripts and API resiliency** — Custom scripts run at multiple points per action (pre-build, post-build, etc.). Tidying unused dependencies and adding retry logic for unreliable external API calls in these scripts improves build consistency and speed.

**Flaky test strategy** — Retrying failed builds due to flaky tests consumes compute. The correct response is to fix flaky tests promptly; see "Author fast and reliable tests for Xcode Cloud" (WWDC22) for guidance.

## APIs & Frameworks

### Xcode Cloud (App Store Connect + Xcode integration)
- **Usage Dashboard** **[NEW]** — App Store Connect > Xcode Cloud > Usage; shows compute minutes, trends, per-product and per-workflow breakdowns
- **Workflow Editor** — Xcode right-click > Edit Workflow; sections: Start Conditions, Actions, Post-Actions
- **Start Conditions**
  - `Scheduled` — time-based trigger (existing)
  - `Branch Changes` — triggers on any new remote commit (existing; recommended over Scheduled)
  - Source Branch: `Any Branch` | `Custom Branches` (exact name or prefix match)
  - Files and Folders: `All Files` | `Custom Conditions` with `Start a Build` / `Don't start a build` rules per folder
- **Actions** — Analyze, Archive, Build, Test; run in parallel per workflow configuration
  - Test action: Destination selector — `Any iOS Simulator Device` | `Recommended iPhones` **[NEW alias]** | specific simulators
- **`ci skip` commit tag** **[NEW]** — append ` ci skip` to a commit message to skip Xcode Cloud processing of that push event
- **Custom Scripts** — shell scripts at `ci_scripts/` path; hooks: `ci_pre_xcodebuild.sh`, `ci_post_xcodebuild.sh`, `ci_post_clone.sh`
- **Build Details** — App Store Connect build overview showing duration, usage, and per-action usage distribution

### App Store Connect
- Xcode Cloud product/workflow management UI
- Usage dashboard with compute minute tracking and trend charts

## Code Highlights

Skipping a CI build via commit message:
```
git commit -m "Update developer documentation ci skip"
```

Branch Changes start condition restricting to release branches (configured in Xcode workflow editor UI, represented conceptually as):
- Source Branch: Custom Branches → begins with `release`
- Files and Folders: Custom Conditions → Don't start a build when Any Folder = `docs`

## Takeaways
- Use the new Xcode Cloud Usage Dashboard in App Store Connect to identify high-consumption workflows and measure the effect of optimizations; usage trending downward confirms best practices are working.
- Replace scheduled start conditions with Branch Changes triggers scoped to the relevant branch prefix, and exclude non-source folders, to eliminate wasted builds.
- Use the "Recommended iPhones" test destination alias to cover meaningful screen-size diversity without running redundant simulators.
- Append `ci skip` to commit messages for documentation or formatting-only commits to prevent unnecessary compute consumption.

---
_Source: WWDC22 Session 110374 page (abstract, transcript, and resource links)._
