# Diagnose unreliable code with test repetitions
**WWDC21 · Session 10296** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10296/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
Xcode 13 adds **test repetitions** to XCTest, providing three modes for running a test multiple times with configurable stopping conditions. This addresses one of the hardest classes of bugs: non-deterministic failures caused by race conditions, global state, environment assumptions, or external service calls—bugs that fail intermittently and are nearly impossible to reproduce with a single test run.

The session walks through a real debugging workflow: a test failing 4 in 100 runs in Xcode Cloud is reproduced locally using "Fixed iterations" mode, then caught in the debugger using "Until failure" with "Pause on Failure" enabled. The root cause—an XCTestExpectation being fulfilled in the wrong completion handler, masking an async race condition—is then fixed by converting to Swift async/await, eliminating the bug class entirely.

## Key Topics

### Three Test Repetition Modes (NEW in Xcode 13)

#### Fixed Iterations (Maximum Repetitions)
- Repeats a test a specified number of times regardless of pass/fail result.
- Use case: understand the reliability of a test suite; reveal tests that fail intermittently; establish a failure rate baseline.
- In Xcode UI: Control-click the test diamond → "Run … Repeatedly…" → set stopping condition to Maximum Repetitions.

#### Until Failure
- Repeats until the test fails.
- Use case: reproduce a non-deterministic failure so you can catch it in the debugger.
- Combine with "Pause on Failure" to halt execution at the point of failure for debugging.

#### Retry on Failure
- Retries the test until it passes, up to a maximum count.
- Use case: identify flaky tests in CI (fail initially but eventually pass); temporarily gather data before fixing.
- Important caveat: retrying failures can hide real product bugs. Use this mode temporarily for diagnosis, not as a permanent CI configuration.

### Enabling Repetitions

**In Xcode UI:**
- Control-click the test diamond next to the test → "Run '\<test\>()' Repeatedly…"
- Set stopping condition and maximum count; optionally enable "Pause on Failure."

**In Test Plans:**
- Configure Test Repetition Mode and Maximum Repetitions in the `.xctestplan` file.
- Allows per-CI-run override without changing code.

**Via xcodebuild CLI:**
- `-test-iterations <N>` — number of iterations (required for all modes).
- `-run-tests-until-failure` — stops on first failure (Until Failure mode).
- `-retry-tests-on-failure` — retries until success (Retry on Failure mode).
- CLI flags override test plan settings.

Example:
```
xcodebuild test -scheme MyApp -test-iterations 100 -run-tests-until-failure
```

### Debugging Workflow Demonstrated
1. Flaky test identified in Xcode Cloud reports (intermittent failure, same assertion every time).
2. Reproduced locally: "Fixed Iterations" × 100 → failed 4 times, confirming non-determinism.
3. "Until Failure" + "Pause on Failure" → caught failure in debugger; inspected `truck` object in LLDB → `flavors == 0` instead of 33.
4. Root cause: `flavorsExpectation.fulfill()` called in the outer completion handler, before `prepareFlavors` inner completion handler returned, so the assertion ran on partially-initialized data.
5. Fix: convert to `async/await` — removes the multi-handler nesting and fulfillment placement entirely.

### Converting Callback-Based XCTest to async/await
- Add `async throws` to the test method signature; Xcode recognizes this automatically.
- Replace `XCTestExpectation` + `wait(for:timeout:)` with `await` at the call site.
- Reduces lines of code, eliminates optional unwrapping, and removes the class of bug caused by `fulfill()` placement.

## APIs & Frameworks

**XCTest (Xcode 13)**
- Test Repetition Mode — three modes: Fixed Iterations, Until Failure, Retry on Failure **[NEW]**
- `XCTestExpectation` — existing async testing mechanism, shown as error-prone for complex multi-handler flows **[existing]**
- `wait(for:timeout:)` — existing wait for expectations **[existing]**
- `async throws` test methods — native Swift concurrency support in XCTest **[NEW in Xcode 13]**
- "Pause on Failure" in test repetition dialog **[NEW]**
- Control-click → "Run '\<Test\>()' Repeatedly…" menu item in Xcode **[NEW]**

**xcodebuild**
- `-test-iterations <N>` flag **[NEW]**
- `-run-tests-until-failure` flag **[NEW]**
- `-retry-tests-on-failure` flag **[NEW]**

**Test Plans (`.xctestplan`)**
- Test Repetition Mode configuration **[NEW]**
- Maximum Repetitions configuration **[NEW]**

**Swift Concurrency**
- `async` / `await` / `throws` in test methods — eliminates completion-handler nesting and expectation fulfillment race conditions **[NEW via Swift 5.5]**

## Code Highlights

Original buggy test (completion handler nesting, wrong fulfillment location):
```swift
func testFlavors() {
    var truck: IceCreamTruck?
    let flavorsExpectation = XCTestExpectation(description: "Get ice cream truck's flavors")
    truckDepot.iceCreamTruck { newTruck in
        truck = newTruck
        newTruck.prepareFlavors { error in
            XCTAssertNil(error)
        }
        flavorsExpectation.fulfill()  // BUG: fulfills before prepareFlavors completes
    }
    wait(for: [flavorsExpectation], timeout: 5)
    XCTAssertEqual(truck?.flavors, 33)
}
```

Fixed with async/await:
```swift
func testFlavors() async throws {
    let truck = await truckDepot.iceCreamTruck()
    try await truck.prepareFlavors()
    XCTAssertEqual(truck.flavors, 33)
}
```

xcodebuild until-failure invocation:
```bash
xcodebuild test -scheme IceCreamTruckCountdown \
    -test-iterations 100 \
    -run-tests-until-failure
```

## Takeaways
- Test repetitions are the right tool for reproducing non-deterministic failures: use Fixed Iterations to establish a failure rate, Until Failure + Pause on Failure to catch the bug in the debugger.
- Retry on Failure is a diagnostic aid for CI, not a permanent fix—it can mask real product bugs; remove it once you have enough data to fix the root cause.
- Converting completion-handler-based async tests to `async/await` (Xcode 13 / Swift 5.5) eliminates the entire class of `XCTestExpectation` placement bugs.
- All three modes are available via xcodebuild CLI flags that override test plan settings, making them easy to apply in CI without code changes.

---
_Source: WWDC21 Session 10296 page (abstract, full transcript, code samples, and resource links)._
