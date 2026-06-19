# Author Fast and Reliable Tests for Xcode Cloud
**WWDC22 · Session 110361** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110361/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
This session focuses on best practices for writing tests that work well in Xcode Cloud's CI/CD environment. Running tests in Xcode Cloud broadens test coverage significantly — multiple devices, OS versions, and runtime analysis tools — but also amplifies unreliability if tests are poorly authored. The session covers two key dimensions: making tests reliable and making them fast.

Reliability techniques include thorough setUp/tearDown, environment variable-controlled test gating (via `XCTSkip`), replacing timeouts with async/await, using `XCTExpectFailure` to handle known transient failures, and leveraging test repetitions to validate new code and diagnose flakiness. Speed improvements come from splitting tests into multiple plans (fast PR-gating plan vs. full scheduled plan), enabling parallel test execution, setting execution time allowances to halt runaway tests, and tuning test repetition counts.

## Key Topics

### Reliable Setup and Teardown
- Use `setUp()` (preferably `async throws`) to establish all state freshly for each test — never assume prior state
- Generate data files in `setUp` using temp directories; don't rely on teardown for inter-test isolation
- For read-only fixtures, check them into the repo; for generated fixtures, use custom Xcode Cloud build scripts

### Environment Variables and XCTSkip
- Xcode Cloud passes environment variables prefixed with `TEST_RUNNER_` to the XCTest runner (prefix stripped in test code)
- Test plans use the variable names without the `TEST_RUNNER_` prefix
- Xcode Cloud UI environment variables take precedence over test plan variables
- Use `ProcessInfo.processInfo.environment["KEY"]` to read variables in tests
- `XCTSkipIf(_:)` / `XCTSkip` — skip tests that don't apply to the current environment

### Async/Await vs. Expectations
- Prefer `async throws` test methods with `await` over `XCTestExpectation` + `wait(for:timeout:)` to avoid timing-related failures
- If using expectations, increase timeout generously for Xcode Cloud where servers may be remote

### XCTExpectFailure
- Marks a known-failing test as an "expected failure" rather than disabling it
- Test still executes; failure is converted to expected failure (suite shows pass)
- Eliminates noise from known transient issues while preserving the test

### Test Repetitions
- Repeat-until-failure mode: validate new code by running repeatedly to detect low-probability failures
- Retry-on-failure mode: work around unreliable external services
- Mock external services where possible for determinism and speed
- Tune `Maximum Test Repetitions` — too high wastes time on real bugs that always fail

### Test Plans and Workflows
- Split tests into a **pull request plan** (fast, key subset, single platform) and a **full suite scheduled plan**
- PR workflow start condition: "Pull Request Changes"
- Scheduled workflow start condition: "On a Schedule for a Branch"
- Each workflow references its own test plan

### Parallel Execution and Execution Time Allowance
- Xcode Cloud runs platforms in parallel by default
- Enable "Execute in parallel" per test bundle in the test plan to run tests concurrently across targets and test classes
- Requires tests to be independent (proper setup/teardown essential)
- "Test Timeouts" in test plan configurations: set maximum execution time allowance (default 600 seconds) to halt runaway tests

## APIs & Frameworks

**XCTest**
- `XCTestCase.setUp() async throws` — async setUp support
- `XCTSkip(_:file:line:)` — throws to skip current test with a message
- `XCTSkipIf(_:_:file:line:)` — conditional skip
- `XCTSkipUnless(_:_:file:line:)` — skip unless condition is true
- `XCTExpectFailure(_:options:failingBlock:)` — mark test as expected to fail
- `XCTestExpectation` — async coordination; `.fulfill()`, `wait(for:timeout:)`
- Test repetition modes (test plan): **Retry on Failure**, **Repeat Until Failure**, **Fixed Number of Repetitions**
- `Maximum Test Repetitions` — test plan setting

**Xcode Cloud Workflow Configuration**
- Start conditions: **Pull Request Changes**, **On a Schedule for a Branch**, branch push, tag
- Actions: **Build**, **Test** (references a test plan)
- Environment variables: set in Workflow editor under "Environment"; `TEST_RUNNER_` prefix for XCTest runner delivery
- Custom build scripts: run pre/post-build steps (e.g., generate fixture data)

**Test Plan Settings**
- "Execute in parallel" — per test bundle option
- "Test Timeouts" + "Execution Time Allowance" — max seconds per test (default 600)
- Test repetition mode + maximum repetitions
- Environment variables section (no `TEST_RUNNER_` prefix needed)

## Code Highlights

Thorough setUp with temp directory fixture:
```swift
override func setUp() async throws {
    let fileURL = FileManager.default.temporaryDirectory
        .appendingPathComponent(UUID().uuidString)
    let data = await mockDonutMenuData()
    try data.write(to: fileURL)
    truck = Truck(menuURL: fileURL)
}
```

Environment variable + XCTSkip + async/await test:
```swift
func testOrderDonut() async throws {
    let host = ProcessInfo.processInfo.environment["BASE_URL"]
    try XCTSkipIf(host == "prod.example.com")
    let donut = try await truck.orderDonut(with: .sprinkles, host: host)
    XCTAssertTrue(donut.hasSprinkles)
}
```

XCTExpectFailure for a known transient failure:
```swift
XCTExpectFailure("https://dev.myco.com/bug/98 Donut ordering service is down")
let donut = try await truck.orderDonut(with: .sprinkles, host: host)
```

## Takeaways
- Always establish test state in `setUp` — never assume a fresh simulator matches any particular prior state.
- Use `TEST_RUNNER_`-prefixed environment variables in Xcode Cloud to gate tests by environment; prefer `XCTSkip` over disabling tests.
- Split workflows into a fast PR-gating plan and a comprehensive scheduled plan to prevent test expansion from slowing down pull requests.
- Enable parallel execution and set execution time allowances to prevent runaway tests from blocking overnight CI runs.

---
_Source: WWDC22 Session 110361 page (abstract, chapter summaries, code samples, and resource links)._
