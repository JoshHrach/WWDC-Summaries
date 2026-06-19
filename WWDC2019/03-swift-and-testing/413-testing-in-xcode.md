# Testing in Xcode
**WWDC19 · Session 413** · [Watch](https://developer.apple.com/videos/play/wwdc2019/413/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, watchOS · Xcode 11

## Overview
This session covers the full testing story in Xcode 11: the XCTest framework for unit and UI tests, the brand-new Test Plans feature for running tests under multiple configurations, and a custom continuous integration pipeline built with `xcodebuild`, `xcresulttool`, and `xccov`. The session targets both developers new to automated testing and teams looking to maximize their test coverage through CI.

## Key Topics

### Testing Pyramid
- **Unit tests** — verify individual functions; fast, numerous, form the pyramid base.
- **Integration tests** — validate clusters of classes working together; fewer, slightly slower.
- **UI tests** — end-to-end, black-box verification of app UI; fewest, slowest, require most maintenance.
- Balance all three layers for thorough, fast coverage.

### XCTest Unit Tests
- Test class: subclass of `XCTestCase`; test methods start with `test`.
- `setUp()` — called before each test; initialize shared objects here.
- `tearDown()` — called after each test; clean up state/data.
- Key assertion APIs:
  - `XCTAssertEqual(_:_:accuracy:_:)` — equality with floating-point tolerance.
  - `XCTAssertThrowsError(_:_:_:)` — verifies an error is thrown; closure inspects the error.
  - `XCTUnwrap(_:_:)` — asserts optional is non-nil and unwraps it; throws on nil (test method must be marked `throws`). **[NEW in Xcode 11]**
- Test diamond in gutter: click to run one test; green = pass, red = fail with highlighted line.
- `continueAfterFailure = false` on UI tests — stop immediately when a failure is encountered.

### XCTest UI Tests
- Drive app via `XCUIApplication`, find elements by type and accessibility identifier.
- `XCUIElement.isHittable` — element both exists and is on screen.
- Debugger + `po app.descendants(matching: .image)` — discover element identifiers at runtime.
- After every UI action, assert the expected screen state (defensive test design).
- Coordinate-independent: mock or simulate location when tests depend on device location.

### Code Coverage
- Enable in scheme or test plan; run tests; results in Report Navigator.
- Source editor gutter shows execution count per line; red = uncovered lines/paths.
- Use coverage gaps to identify where to write more tests.

### Test Plans **[NEW in Xcode 11]**
- A test plan is a `.xctestplan` JSON file that can be shared across multiple schemes.
- Defines **Shared Settings** (common to all configurations) and one or more **Test Configurations** (each = a single full run of all tests with its own options).
- Each configuration has a unique, meaningful name (appears in test diamond popover and test report).
- Options per configuration include: language/locale, test execution order (alphabetical vs. random), sanitizers (address, thread, undefined behavior), environment variables, command-line arguments, memory diagnostics (zombie objects), screenshot/attachment retention, code coverage toggle, UI test parallelization.
- **Adopting Test Plans:** Edit scheme → Test action → "Convert to use Test Plans".
- Running: click test diamond runs all configurations; option-click to run a single configuration.
- Test report shows per-configuration results — a test can pass in one configuration and fail in another ("Mixed" status filter).
- **Localization screenshots** — new option: preserves screenshots from all test runs (even passing) for use as App Store screenshots and localization review. **[NEW]**
- Multiple test plans per scheme: default plan runs unless overridden with `xcodebuild -testPlan <name>`.

### Example Test Plan Configurations
1. Memory safety: Address Sanitizer + Zombie Objects.
2. Concurrency: Thread Sanitizer + UBSan + random test order.
3. Multi-locale: US English + German + Japanese (with localization screenshots).
4. Diagnostics: custom environment variable for verbose logging + keep all attachments.

### Continuous Integration with xcodebuild
Two workflows:
- **Build + test in one step:** `xcodebuild test -project ... -scheme ... -destination ...`
- **Build then test separately (two machines):**
  1. `xcodebuild build-for-testing` → produces build products + `.xctestrun` file.
  2. `xcodebuild test-without-building -xctestrun <path> -destination ...`
- `-resultBundlePath <path>` — save structured test results to a bundle for later analysis.
- `-testPlan <name>` — override the default test plan when a scheme has multiple plans.
- `xcodebuild -showTestPlans` — list all test plans in a scheme.
- Multiple `-destination` flags — run on several simulators/devices simultaneously.

### Result Bundles **[NEW format in Xcode 11]**
- Produced by passing `-resultBundlePath` to `xcodebuild`.
- New format in Xcode 11: 4× smaller on disk, openable directly in Xcode, and programmatically queryable.
- Contains: build log, test report, code coverage report, test attachments.
- Double-click in Finder to open in Xcode's report UI.

### xcresulttool **[NEW in Xcode 11]**
- New command-line tool for programmatic access to result bundle data.
- `xcresulttool get --path <bundle>` — emits full JSON of build + test results.
- JSON schema is **publicly documented and versioned** — stable API for CI automation.
- `xcresulttool formatDescription` — prints the JSON schema for reference.
- Extract build failures: nested in JSON with message, file path, and line number.
- Extract test failures: nested with test name and assertion message.

### xccov
- `xccov view <bundle>` — human-readable or JSON code coverage report per target/file/method.
- `xccov diff <bundle1> <bundle2>` — compare coverage between two result bundles (shows +/- percentage per file).
- Man page: `man xccov`.

## APIs & Frameworks

### XCTest
- `XCTestCase` — base class for all test classes
- `setUp()` / `tearDown()` — per-test setup and cleanup
- `XCTAssertEqual(_:_:_:file:line:)` — equality assertion
- `XCTAssertEqual(_:_:accuracy:_:file:line:)` — floating-point equality with tolerance
- `XCTAssertThrowsError(_:_:file:line:_:)` — error throwing assertion with inspection closure
- `XCTUnwrap(_:_:file:line:)` — unwrap optional or fail **[NEW]**
- `XCUIApplication` — proxy for the app under UI test
- `XCUIElement` — UI element query result
- `XCUIElement.isHittable` — existence + on-screen check
- `XCTAttachment` — file/data attachment from tests
- `continueAfterFailure: Bool` — stop on first UI test failure

### Xcode 11 Test Plan Format
- `.xctestplan` — JSON file checked into source control
- Shared Settings + Test Configurations array
- `localizationScreenshotsEnabled` — collect screenshots for localization **[NEW]**

### Command-Line Tools
- `xcodebuild test` — build and test
- `xcodebuild build-for-testing` — build only, produce `.xctestrun`
- `xcodebuild test-without-building -xctestrun` — test using pre-built products
- `xcodebuild -resultBundlePath` — write result bundle
- `xcodebuild -testPlan` — specify test plan by name **[NEW]**
- `xcodebuild -showTestPlans` — list test plans **[NEW]**
- `xcresulttool get` — extract result bundle JSON **[NEW]**
- `xcresulttool formatDescription` — show JSON schema **[NEW]**
- `xccov view` — view coverage report
- `xccov diff` — diff two coverage reports

## Code Highlights

Unit test with XCTUnwrap and tolerance:
```swift
class DistanceCalculatorTests: XCTestCase {
    var calculator: DistanceCalculator!
    
    override func setUp() {
        super.setUp()
        calculator = DistanceCalculator()
    }
    
    func testCoordinatesOfSeattle() throws {
        let city = try XCTUnwrap(calculator.city("Seattle"))
        XCTAssertEqual(city.coordinates.latitude, 47.6062, accuracy: 0.001)
        XCTAssertEqual(city.coordinates.longitude, -122.3321, accuracy: 0.001)
    }
    
    func testSanFranciscoToNewYork() throws {
        let distance = try calculator.distanceInMiles(from: "San Francisco", to: "New York")
        XCTAssertEqual(distance, 2572, accuracy: 1)
    }
    
    func testUnknownCityThrows() {
        XCTAssertThrowsError(try calculator.distanceInMiles(from: "Cupertino", to: "Paris")) { error in
            XCTAssertEqual(error as? CalculatorError, .unknownCity)
        }
    }
}
```

UI test with element discovery and state verification:
```swift
class DiscoverUITests: XCTestCase {
    let app = XCUIApplication()
    
    override func setUp() {
        super.setUp()
        continueAfterFailure = false
        app.launch()
    }
    
    func testMilesToParis() {
        app.tabBars.buttons["Discover"].tap()
        XCTAssertTrue(app.staticTexts["San Francisco"].isHittable)
        
        let sfImage = app.images["San Francisco, image"]
        sfImage.swipeLeft()
        
        XCTAssertTrue(app.staticTexts["Paris"].isHittable)
        XCTAssertTrue(app.staticTexts["5586 miles"].isHittable)
    }
}
```

CI: build then test on separate machines:
```bash
# Machine 1: build
xcodebuild build-for-testing \
  -project MyApp.xcodeproj \
  -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 11'

# Machine 2: test (copy build products and xctestrun first)
xcodebuild test-without-building \
  -xctestrun MyApp.xctestrun \
  -destination 'platform=iOS Simulator,name=iPhone 11' \
  -resultBundlePath results.xcresult

# Extract failures for issue tracker
xcresulttool get --path results.xcresult

# Compare coverage
xccov diff before.xcresult after.xcresult
```

## Takeaways
- Organize tests as unit → integration → UI using the pyramid model; XCTest supports all three with the same authoring tools and test diamond workflow.
- **Test Plans** (new in Xcode 11) replace duplicated schemes: define all test variation (language, sanitizers, execution order, environment) in one `.xctestplan` file shared across schemes.
- `XCTUnwrap` (new) replaces `XCTAssertNotNil` + force-unwrap pattern, propagating a clean failure when an optional is nil.
- The new result bundle format with `xcresulttool` and `xccov` gives CI pipelines a documented, stable JSON API for extracting failures and tracking coverage trends over time.

---
_Source: WWDC19 Session 413 page (abstract, full transcript, and resource links)._
