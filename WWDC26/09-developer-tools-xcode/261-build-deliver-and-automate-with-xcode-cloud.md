# Build, Deliver, and Automate with Xcode Cloud

**WWDC26 · Session 261** · [Watch](https://developer.apple.com/videos/play/wwdc2026/261/)

_Platforms:_ Xcode Cloud · Xcode 27 · iOS · macOS · iPadOS · tvOS · watchOS · visionOS

## Overview

This session provides both an introduction to Xcode Cloud for new users and a tour of advanced automation capabilities for experienced teams. At its core, Xcode Cloud builds and tests apps in parallel across multiple devices and OS versions in Apple's cloud infrastructure, catching bugs and performance issues before they reach customers and delivering directly to TestFlight and the App Store.

The new onboarding flow dramatically lowers the barrier to entry: developers can connect a source repository and start an automatic build-and-test workflow with just a few clicks, without needing to understand all of Xcode Cloud's concepts upfront. Distribution configuration for TestFlight is similarly streamlined — creating an App Store Connect app record and enabling TestFlight delivery from a new workflow is guided step-by-step from within Xcode.

For teams needing custom integrations, the session covers webhooks (which deliver build event payloads to external services) and additional repository support (which gives builds access to shared framework dependencies stored in separate Git repos). These two features unlock sophisticated automation scenarios like custom dashboards, Slack notifications, and monorepo-style multi-repo dependency management.

## Key Topics

### Essential Concepts
- Xcode Cloud builds and tests across multiple devices and OS versions in parallel.
- Triggered automatically on every commit to the connected source repository.
- Delivers results to TestFlight and App Store with no additional configuration.

### Getting Started (New Onboarding Flow)
- **[NEW]** Streamlined onboarding assistant: connect a source repository to start builds.
- Supports iOS and macOS multi-platform apps simultaneously.
- Automatic build and test workflow created on first setup.
- No upfront knowledge of all Xcode Cloud concepts required.

### Distribution
- Create an App Store Connect app record for the product.
- Configure a distribution workflow to enable TestFlight delivery directly from Xcode.
- App Store submission available from the same workflow configuration.

### Webhooks
- Configure webhook URLs on the Xcode Cloud product.
- Build events (started, succeeded, failed) POST a JSON payload to the configured URL.
- Use cases: custom CI dashboards, Slack/Teams notifications, downstream automation triggers.

### Additional Repositories
- Add extra Git repositories to a product so builds can access shared dependencies.
- Enables teams that split frameworks or packages into separate repos from their main app repo.
- All repositories cloned into the build environment alongside the primary repo.

## APIs & Frameworks

**Xcode Cloud**
- **[NEW]** Onboarding assistant (connect repository, create first workflow in minutes)
- Workflows — the primary configuration unit (triggers, build actions, test actions, distribution)
- Build actions — compile the app for target platforms and SDKs
- Test actions — run unit and UI tests across device/OS matrix
- Distribute actions — push to TestFlight, App Store, or internal distribution
- **[NEW]** Simplified TestFlight distribution workflow configuration from Xcode
- Webhooks — HTTP POST on build events (started / succeeded / failed)
- Webhook payload: JSON containing build number, status, branch, commit, product info
- Additional repositories — attach supplemental Git repos to a product
- App Store Connect integration — create app records, manage TestFlight groups

**Source Control Integration**
- GitHub, GitLab, Bitbucket, self-hosted Git (SSH)
- Branch, tag, and pull request triggers

**Related Documentation**
- [Extend your Xcode Cloud workflows (WWDC24)](https://developer.apple.com/videos/play/wwdc2024/10200)
- [Create practical workflows in Xcode Cloud (WWDC23)](https://developer.apple.com/videos/play/wwdc2023/10278)

## Code Highlights

No code samples in this session — it is a tooling and configuration walkthrough. Webhook integration is configured through the Xcode Cloud product settings UI and receives standard HTTP POST payloads; no SDK API is required.

## Takeaways

- The new onboarding assistant reduces Xcode Cloud setup from a multi-hour process to minutes — any developer with a source repository can have CI running on first commit.
- Webhooks are the primary extension point for integrating Xcode Cloud into custom team tooling without needing the App Store Connect API.
- Additional repositories solve the most common multi-repo pain point: shared framework dependencies — without requiring Swift Package Manager proxies or git submodules.
- Xcode Cloud's parallel device matrix approach catches device-specific bugs that single-device local testing misses.

---
_Source: WWDC26 Session 261 page (abstract, chapter summaries, and resource links)._
