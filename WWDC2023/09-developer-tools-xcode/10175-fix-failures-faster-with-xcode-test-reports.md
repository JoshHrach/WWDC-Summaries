# Fix Failures Faster with Xcode Test Reports
**WWDC23 · Session 10175** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10175/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, visionOS 1, watchOS 10

## Overview
Xcode 15 introduces a redesigned Test Report that makes it faster to understand test run results, identify failure patterns, and navigate to root causes. The new report is available both in Xcode's local IDE and in Xcode Cloud, providing a consistent debugging workflow whether tests are run manually or in CI.

The report is built around clear organizational concepts — test methods, classes, bundles, and plans, each combined with configurations and run destinations to produce a matrix of test method runs. The redesigned UI surfaces a high-level summary, intelligent failure grouping via Insights, and a new interactive UI test debugging workflow combining video playback, a timeline of test activity, and an Automation Explorer.

The Automation Explorer provides frame-accurate video playback synchronized with test activity events, bounding box overlays of UI elements at the moment of failure, and element hierarchy information — giving developers a significantly richer understanding of why UI tests fail.

## Key Topics

### Test Structure Concepts
- Test methods: individual test functions producing pass/fail/skip/expected-failure results.
- Test classes: groups of related test methods.
- Test bundles: one or more test classes; each bundle is either Unit or UI.
- Test plans: one or more bundles; support multiple configurations (language, locale, code coverage, repetition count) and multiple run destinations.
- Test method run: a unique result instance for every combination of test method, configuration, and run destination.

### Test Report Summary
- High-level view of environment details, configuration traits, and run destination summary.
- Heat map showing pass/fail status across all device × configuration combinations at a glance.
- Quick access to all failed tests directly from the summary view.

### Insights
- Xcode analyzes results across all configurations and run destinations to surface patterns.
- "Common Failure Patterns" insight: groups tests with similar failure messages, helping identify systemic issues quickly.
- "Longest Test Runs" insight: flags the slowest tests to help optimize test suite performance.

### Test Details View
- Dedicated per-test view showing header with all configurations/run destinations.
- For unit tests: failure message, call stack with jump-to-source navigation.
- For UI tests: Activities tab with test activity timeline, Automation Explorer, and scrubber.

### Automation Explorer and UI Test Debugging
- Video playback: synchronized frame-accurate replay of UI tests.
- Test activity timeline: ordered list of every event (taps, swipes, orientation changes) during the test run.
- Scrubber: linear representation of the test run with failure markers and orientation change indicators.
- Clicking an activity event updates the Automation Explorer to the corresponding video frame.
- Bounding boxes overlay UI elements on-screen at the moment of failure; clicking a bounding box reveals identifier and hierarchy information for that element.

## APIs & Frameworks
- Xcode 15 Test Report **[NEW]** — redesigned test results viewer for local and Xcode Cloud runs
- Xcode Cloud build overview — workflow, code change, and action status summary
- Test Plans (`.xctestplan`) — multi-configuration test execution plans
- Test Configurations — language, locale, code coverage, and repetition settings within test plans
- Run Destinations — devices and simulators where tests execute
- Xcode Cloud Workflow — CI/CD pipeline that triggers test report generation
- UI Test Automation Explorer **[NEW]** — frame-accurate video playback with element overlays for UI test failures
- Test activity timeline **[NEW]** — ordered event log synchronized with video playback
- Test scrubber **[NEW]** — linear test run navigator with failure and orientation markers
- Common Failure Patterns Insight **[NEW]** — automatic grouping of tests with similar failure messages
- Longest Test Runs Insight **[NEW]** — identification of slowest tests in the suite
- Heat map **[NEW]** — visual grid of pass/fail counts per device × configuration
- XCTest framework — underlying unit and UI test framework
- XCUITest — UI testing component of XCTest

## Code Highlights
This session is UI/tooling focused; no code samples are provided. The key interactions are entirely within the Xcode 15 IDE and Xcode Cloud web UI.

## Takeaways
- The redesigned Test Report surfaces failure patterns (Insights) immediately, so developers know where to start investigating rather than scanning through long result lists.
- The Automation Explorer with synchronized video playback and element bounding boxes dramatically reduces the time needed to diagnose UI test failures.
- Insights — Common Failure Patterns and Longest Test Runs — help identify systemic issues and performance bottlenecks across the full configuration × destination matrix.
- The same enriched Test Report is available in both local Xcode runs and Xcode Cloud CI, providing a consistent experience across local and automated testing workflows.

---
_Source: WWDC23 Session 10175 page (abstract, chapter summaries, code samples, and resource links)._
