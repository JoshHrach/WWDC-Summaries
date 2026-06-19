# Testing Tips & Tricks
**WWDC18 · Session 417** · [Watch](https://developer.apple.com/videos/play/wwdc2018/417/)

_Platforms:_ iOS 12, macOS Mojave 10.14

## Overview
This session shares four concrete techniques for writing high-quality, fast, and maintainable XCTest unit tests — grounded in the experience of building a real points-of-interest app. The techniques cover testing networking code using `URLProtocol` mocking, isolating `NotificationCenter` usage with dependency injection, mocking external SDK classes (including delegates) using protocols, and eliminating artificial delays from tests that involve timers or asynchronous work.

The session opens with a recap of the testing pyramid: a large base of fast, focused unit tests; a middle tier of integration tests covering subsystems; and a small top tier of end-to-end UI tests. Each technique is presented as a way to push work down to the appropriate tier.

## Key Topics

### Testing Networking Code
- Decompose view controller networking into small, pure functions: request preparation (`makeRequest`) and response parsing (`parseResponse`)
- Pure functions with no side effects are trivially unit-testable
- For the URLSession integration layer, use `URLProtocol` subclassing to intercept and mock network requests in a test bundle
- `MockURLProtocol`: override `canInit(with:)` to claim all requests, use `requestHandler` closure to let tests inspect outgoing requests and return stub responses
- Configure the `URLSession` with a `URLSessionConfiguration` that registers the mock protocol via `protocolClasses`
- Integration tests using this technique verify that `task.resume()` is actually called, that request formation is correct, and that response parsing works end-to-end
- End-to-end UI tests: use a local mock server; also include a few unit tests that call directly into the network stack against the real server to verify API contract

### Testing Foundation Notifications (Isolation via Dependency Injection)
- Notifications posted to `NotificationCenter.default` can trigger unintended side effects in other parts of the app or framework code, causing flaky tests
- Fix: inject a separate `NotificationCenter` instance instead of using `.default`
- Add `notificationCenter: NotificationCenter = .default` as a defaulted initializer parameter — existing production code requires no changes; tests pass a fresh instance
- Testing observation: post the notification to the injected center in the test body
- Testing posting: use `XCTNSNotificationExpectation(name:object:notificationCenter:)` with `timeout: 0` (expectation should already be fulfilled when the method returns)

### Mocking External SDK Classes with Protocols
- Problem: external SDK classes (e.g., `CLLocationManager`) can't always be created in tests, may trigger permission dialogs, and subclassing is risky
- Solution: define a protocol that captures exactly the methods and properties your code uses from the external class
- Create an empty `extension ExternalClass: YourProtocol { }` — the class already satisfies the requirements
- Replace all usage of the concrete type with the protocol type in your class; inject via a defaulted initializer parameter
- Mocking delegates: define a parallel mock delegate protocol with the same method signatures but replacing the concrete manager parameter type with your mock protocol type; implement the real delegate conformance to forward calls to the mock delegate

### Test Execution Speed
- **Avoid `NSPredicateExpectation`** in unit tests — it uses polling and is significantly slower; use `XCTestExpectation`, `XCTNSNotificationExpectation`, or `XCTKVOExpectation` instead
- **Eliminate timer/async delays**: break `Timer.scheduledTimer` (which both creates and schedules) into create + `RunLoop.add(_:forMode:)` separately; extract the scheduling step behind a protocol (`TimerScheduler`) and inject a mock implementation in tests that fires the timer immediately
- Mock timer approach: `MockTimerScheduler` receives the timer in `handleAddTimer` closure, records its interval for assertions, then calls `timer.fire()` to bypass the delay entirely
- **Speed up app launch in test targets**: detect test environment via environment variable (set in scheme's Test action arguments: `IS_UNIT_TESTING=1`); in `applicationDidFinishLaunching`, skip non-essential setup (view controllers, network requests, analytics) when this flag is set

## APIs & Frameworks

**XCTest**
- `XCTestCase` — base class for test cases; methods marked `throws` can use `try` without explicit do-catch
- `XCTestExpectation` — manual expectation; `fulfill()` to satisfy, `wait(for:timeout:)` to block
- `XCTNSNotificationExpectation(name:object:notificationCenter:)` — expectation fulfilled when a notification is received on a specific center
- `XCTKVOExpectation` — expectation fulfilled via Key-Value Observing
- `XCTAssertEqual`, `XCTAssertTrue`, `XCTAssertNil`, etc. — assertion functions

**Foundation Networking**
- `URLProtocol` — abstract base class for URL loading system interceptors; subclass in test bundle to mock network requests
- `URLProtocol.canInit(with:)` — class method; return `true` to handle a request
- `URLProtocol.canonicalRequest(for:)` — return the canonical form of the request
- `URLProtocol.startLoading()` / `stopLoading()` — implement to send mock response via `client`
- `URLProtocolClient` — protocol for communicating response data back to the URL loading system
  - `urlProtocol(_:didReceive:cacheStoragePolicy:)` — send mock response headers
  - `urlProtocol(_:didLoad:)` — send mock body data
  - `urlProtocolDidFinishLoading(_:)` — signal completion
  - `urlProtocol(_:didFailWithError:)` — signal failure
- `URLSessionConfiguration.protocolClasses` — register custom `URLProtocol` subclass
- `JSONDecoder` — used in response parsing; `Decodable` protocol conformance on model types

**Foundation**
- `NotificationCenter` — `default` class property plus the ability to create separate instances for isolation
- `NotificationCenter.addObserver(forName:object:queue:using:)` — block-based observation
- `NotificationCenter.post(name:object:userInfo:)` — post a notification
- `RunLoop` — `add(_:forMode:)` for adding timers; abstract behind `TimerScheduler` protocol for testability
- `Timer` — `scheduledTimer(withTimeInterval:repeats:block:)` vs. `init(timeInterval:repeats:block:)` + manual scheduling

**CoreLocation (as example of external SDK mocking)**
- `CLLocationManager` — mocked via `LocationFetcher` protocol
- `CLLocationManagerDelegate` — mocked via `LocationFetcherDelegate` protocol

## Code Highlights

Registering a mock URLProtocol:
```swift
class MockURLProtocol: URLProtocol {
    static var requestHandler: ((URLRequest) throws -> (HTTPURLResponse, Data))?
    override class func canInit(with request: URLRequest) -> Bool { return true }
    override class func canonicalRequest(for request: URLRequest) -> URLRequest { return request }
    override func startLoading() {
        guard let handler = MockURLProtocol.requestHandler else { return }
        do {
            let (response, data) = try handler(request)
            client?.urlProtocol(self, didReceive: response, cacheStoragePolicy: .notAllowed)
            client?.urlProtocol(self, didLoad: data)
            client?.urlProtocolDidFinishLoading(self)
        } catch {
            client?.urlProtocol(self, didFailWithError: error)
        }
    }
    override func stopLoading() {}
}

// In test setup:
let config = URLSessionConfiguration.ephemeral
config.protocolClasses = [MockURLProtocol.self]
let session = URLSession(configuration: config)
```

Isolating NotificationCenter:
```swift
// Production code:
class MyViewController: UIViewController {
    let notificationCenter: NotificationCenter
    init(notificationCenter: NotificationCenter = .default) {
        self.notificationCenter = notificationCenter
    }
}

// Test:
let center = NotificationCenter()
let vc = MyViewController(notificationCenter: center)
center.post(name: .authChanged, object: nil)
```

Mocking a timer scheduler:
```swift
protocol TimerScheduler {
    func add(_ timer: Timer, forMode mode: RunLoop.Mode)
}
extension RunLoop: TimerScheduler {}

struct MockTimerScheduler: TimerScheduler {
    var handleAddTimer: ((Timer) -> Void)?
    func add(_ timer: Timer, forMode mode: RunLoop.Mode) {
        handleAddTimer?(timer)
    }
}

// In test:
var mock = MockTimerScheduler()
mock.handleAddTimer = { timer in
    XCTAssertEqual(timer.timeInterval, 10.0)
    timer.fire()  // bypass delay entirely
}
```

## Takeaways
- Use `URLProtocol` subclassing to mock network calls at the integration test level without changing production networking code — and catch bugs like forgetting to call `task.resume()`.
- Inject `NotificationCenter` instances via defaulted initializer parameters to keep tests fully isolated from the rest of the app's notification traffic.
- Protocols with conforming extensions on external types are safer and more compiler-enforced than subclassing for mocking SDK classes.
- Never sleep or wait a fixed number of seconds in tests — always extract the delay mechanism and mock it so tests fire immediately without depending on CPU scheduling.

---
_Source: WWDC18 Session 417 page (abstract, full transcript, and resource links)._
