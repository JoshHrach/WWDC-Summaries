# Get Your Test Results Faster
**WWDC20 · Session 10221** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10221/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session focuses on shortening the testing feedback loop through two new Xcode 12 capabilities: Execution Time Allowance and Parallel Distributed Testing on physical devices. The core problem it addresses is test suites that hang indefinitely, never producing results — a situation that breaks the write-run-interpret feedback cycle entirely.

The session walks through a real-world deadlock scenario where a helper method acquired the same lock as the method under test. Enabling Execution Time Allowance caused the hang to be caught, killed, and diagnosed with an attached spindump, which pinpointed the deadlock in a few seconds. The second half covers how to distribute tests across multiple physical iOS/tvOS devices simultaneously in Xcode 12 — a new capability that produced a 30% speedup for XCTest's own test suite with just two devices.

## Key Topics

**Execution Time Allowance**
- New test plan option in Xcode 12 **[NEW]** — enforces a per-test time limit
- When a test exceeds its allowance: Xcode captures a spindump, kills the hung test, restarts the test runner, continues the remaining suite
- Default allowance: 10 minutes per test; timer resets on each test completion
- Values are rounded to the nearest minute (minimum: 60 seconds)
- Spindump is attached to the test result in the report, double-clickable to open in editor

**Precedence Order for Time Allowances**
1. `executionTimeAllowance` API on `XCTestCase` (highest)
2. `xcodebuild -test-timeouts-enabled YES -default-test-execution-time-allowance <seconds>`
3. Test plan configuration setting
4. System default (10 minutes, lowest)

**Maximum Allowance Enforcement**
- Can enforce an absolute ceiling via test plan or `xcodebuild -maximum-test-execution-time-allowance <seconds>`
- Prevents any test from requesting unlimited time, regardless of API or configuration settings

**Spindump for Diagnosing Hangs**
- Shows which functions each thread is spending time in (sampled stack traces)
- Useful for identifying deadlocks, spinlocks, and excessive main-thread work
- Can be manually captured: `spindump` in Terminal or via Activity Monitor
- Key workflow: find test method name in spindump → trace stack up to lock acquisition

**Parallel Distributed Testing**
- Xcode 10+ feature; Xcode 12 extends it to physical iOS and tvOS devices **[NEW]**
- Xcodebuild distributes test classes non-deterministically across destinations
- Each device runs one test class at a time; receives a new class when finished
- 30% speedup achieved on XCTest's own suite with just two physical devices

**Test Parallelization Options**
- Parallel Distributed Testing — splits test classes across multiple run destinations; fastest for independent tests
- Parallel Destination Testing — runs the entire suite on each destination; best for cross-OS/device compatibility verification

**Recommendations**
- Use Execution Time Allowances specifically to guard against hangs, not as a performance regression tool (use XCTest performance APIs for regressions, Instruments for profiling)
- Use identical devices and OS versions in a distributed device pool to avoid non-deterministic destination-specific failures
- For device/OS-specific tests, use Parallel Destination Testing (not Distributed) to ensure each configuration runs the full suite

## APIs & Frameworks

### XCTest — Execution Time Allowance (New)
- `XCTestCase.executionTimeAllowance: TimeInterval` **[NEW]** — override per-test or per-class time allowance
  - Values rounded to nearest minute; minimum 60 seconds
  - Takes highest precedence over all other allowance settings
  - Set in `setUp()` or directly on the property in the test class

### Xcodebuild — New Flags (New)
- `-test-timeouts-enabled YES` **[NEW]** — enables Execution Time Allowance for the test run
- `-default-test-execution-time-allowance <seconds>` **[NEW]** — sets default allowance for all tests
- `-maximum-test-execution-time-allowance <seconds>` **[NEW]** — hard cap regardless of other settings
- `-parallel-testing-enabled YES` — enables Parallel Distributed Testing
- `-parallelize-tests-among-destinations` — distributes test classes across the specified destinations
- `-destination` — specify multiple run destinations for parallel distribution

### Test Plans (Xcode 12)
- "Test Timeouts" toggle **[NEW]** — enables Execution Time Allowance in test plan configuration
- "Default Test Execution Time Allowance" **[NEW]** — test plan-level default per-test limit
- "Maximum Test Execution Time Allowance" **[NEW]** — absolute ceiling enforced for all tests in the plan

### XCTest — Existing APIs Referenced
- `XCTestCase` — base class; `executionTimeAllowance` property added
- `XCTAssertNotEqual(_:_:)` — assertion used in session example
- XCTest performance APIs — `measure {}` for regression testing (recommended instead of allowances for perf)

## Code Highlights

Setting a custom time allowance for a specific test class:
```swift
class SmoothieNetworkingTests: XCTestCase {
    override var executionTimeAllowance: TimeInterval {
        return 60 * 5  // 5 minutes — rounded to nearest minute
    }

    func testUpdatingSmoothiesFromServer() throws {
        let originalSmoothies = Smoothie.all
        try Smoothie.fetchSynchronouslyFromServer()
        XCTAssertNotEqual(originalSmoothies, Smoothie.all)
    }
}
```

Enabling parallel distributed testing via xcodebuild:
```bash
xcodebuild test \
  -project Fruta.xcodeproj \
  -scheme Fruta \
  -parallel-testing-enabled YES \
  -parallelize-tests-among-destinations \
  -destination 'platform=iOS,id=DEVICE_ID_1' \
  -destination 'platform=iOS,id=DEVICE_ID_2'
```

Enabling time allowances via xcodebuild:
```bash
xcodebuild test \
  -project Fruta.xcodeproj \
  -scheme Fruta \
  -test-timeouts-enabled YES \
  -default-test-execution-time-allowance 600 \
  -maximum-test-execution-time-allowance 1200
```

## Takeaways
- Enable Execution Time Allowance in every CI test plan — without it, a single deadlocked test can silently block the entire suite forever, producing zero results.
- The spindump attached to a timed-out test result is the fastest path to diagnosing a deadlock: search for the test method name in the spindump and trace the stack up to the blocking call.
- Time allowance values are rounded to the nearest minute; set them with this in mind and use the maximum allowance setting to prevent runaway configurations.
- Parallel Distributed Testing on physical devices is new in Xcode 12 — add a second device to your CI pool and expect around a 30% speedup immediately, with greater gains as the pool grows.

---
_Source: WWDC20 Session 10221 page (abstract, transcript, code samples, and resource links)._
