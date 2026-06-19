# Combine in Practice
**WWDC19 · Session 721** · [Watch](https://developer.apple.com/videos/play/wwdc2019/721/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
This session is the practical companion to "Introducing Combine" (Session 722). It teaches engineers how to use Combine operators to handle errors gracefully, chain complex asynchronous transformations with `flatMap`, schedule work on specific threads/queues, and integrate Combine into real app code including SwiftUI bindings and UIKit form validation.

The session walks through two extended examples: a notification-driven magic-trick downloader that demonstrates operator composition and error recovery patterns, and a wizard school sign-up form that shows how to combine asynchronous network validation with synchronous local validation using `CombineLatest`, `@Published`, `debounce`, `removeDuplicates`, `Future`, and `assign(to:on:)`.

## Key Topics

**Publisher / Subscriber Recap**
- Publishers specify `Output` and `Failure` associated types; Subscribers specify `Input` and `Failure`
- Subscription lifecycle: `receive(subscription:)` once → `receive(_:)` zero or more times → `receive(completion:)` at most once
- A completion of `.finished` or `.failure` terminates the stream; no further values can arrive

**Error Handling Operators**
- `assertNoFailure()` — asserts the stream never fails; traps at runtime if it does; changes failure type to `Never`
- `retry(_:)` — re-subscribes to the upstream publisher N times on failure
- `mapError(_:)` — transforms an upstream error to a different error type
- `catch(_:)` — replaces the entire upstream with a recovery publisher when failure arrives; terminates connection to original upstream
- `replaceError(with:)` — shorthand catch that substitutes a single fallback value

**`flatMap` for Per-Value Nested Publishers**
- `flatMap(maxPublishers:_:)` — maps each upstream value to a new publisher and subscribes to it; inner publisher's values flow downstream
- Use `flatMap` + `catch` (nested) to recover from inner errors while keeping the outer stream alive — the outer upstream is not terminated on inner failure
- This pattern is the correct way to perform per-element error recovery in an ongoing stream (e.g., decoding each notification independently)

**`Just` and `Future`**
- `Just(_:)` — publishes a single value synchronously then completes; failure type is `Never`; used as a recovery or placeholder publisher
- `Future<Output, Failure>(_:)` — wraps a single asynchronous operation (callback-based API) as a publisher; closure receives a `Promise` closure; call `promise(.success(value))` or `promise(.failure(error))`

**Scheduled Operators**
- `delay(for:scheduler:)` — defers delivery of every element by a time interval
- `throttle(for:scheduler:latest:)` — delivers at most one value per time window
- `debounce(for:scheduler:)` — waits for a quiet period before forwarding the most recent value; ideal for search/text-field network calls
- `receive(on:)` — guarantees downstream events are delivered on a specific `Scheduler` (e.g., `DispatchQueue.main`, `RunLoop.main`)
- `subscribe(on:)` — controls which scheduler upstream work is performed on

**`removeDuplicates`**
- Suppresses consecutive identical values; prevents redundant network calls when text input returns to the same value within a debounce window

**`@Published` Property Wrapper**
- `@Published var property: T` — adds a `Publisher` to any stored property; access via `$property` (the projected value)
- Setting the property sends the new value to all subscribers
- Failure type is `Never`; Output type matches the property's type

**`CombineLatest`**
- `CombineLatest(_:_:transform:)` — combines the latest values from two publishers whenever either emits; transform closure receives both current values
- `CombineLatest3`, `CombineLatest4` for three/four publishers
- Output emits only after all upstream publishers have each emitted at least once

**Subscribers**
- `sink(receiveCompletion:receiveValue:)` — closure-based subscriber; returns `AnyCancellable`
- `assign(to:on:)` — key-path assignment subscriber; directly sets a property on an object; returns `AnyCancellable`
- Subjects: `PassthroughSubject<Output, Failure>` — no stored value; `CurrentValueSubject<Output, Failure>` — stores and replays last value
  - Both are Publisher + Subscriber hybrids; support `send(_:)` for imperative value injection
  - `share()` operator injects a `PassthroughSubject` for multicasting

**`eraseToAnyPublisher`**
- Wraps any publisher in `AnyPublisher<Output, Failure>`, hiding implementation details at API boundaries
- Recommended at module/type boundaries to prevent leaking internal operator chain types

**`AnyCancellable`**
- Wraps a `Cancellable`; automatically calls `cancel()` on `deinit`
- Store in an `ivar` or `Set<AnyCancellable>` to keep subscriptions alive for the object's lifetime

**SwiftUI Integration (`BindableObject`)**
- Conform model types to `BindableObject` (WWDC19-era API; later renamed `ObservableObject`)
- Declare `var didChange: PassthroughSubject<Void, Never>` — call `didChange.send()` in property observers
- Use `@ObjectBinding` in SwiftUI views to auto-subscribe and re-render on model changes

## APIs & Frameworks

### Combine (NEW)
- `Publisher` protocol **[NEW]** — `associatedtype Output`, `associatedtype Failure: Error`
- `Subscriber` protocol **[NEW]** — `receive(subscription:)`, `receive(_:)`, `receive(completion:)`
- `Subscription` protocol **[NEW]** — `request(_:)`, `cancel()`
- `AnyCancellable` **[NEW]** — auto-cancelling wrapper; `cancel()`
- `AnyPublisher<Output, Failure>` **[NEW]** — type-erased publisher
- `Just<Output>` **[NEW]** — single-value synchronous publisher
- `Future<Output, Failure>` **[NEW]** — single async value via promise closure
- `PassthroughSubject<Output, Failure>` **[NEW]** — no stored value, multicast
- `CurrentValueSubject<Output, Failure>` **[NEW]** — stores last value, replays to new subscribers
- `Publisher.map(_:)` **[NEW]** — synchronous transform
- `Publisher.tryMap(_:)` **[NEW]** — throwing transform; failure type becomes `Error`
- `Publisher.decode(type:decoder:)` **[NEW]** — decode `Decodable` types from `Data` publishers
- `Publisher.flatMap(maxPublishers:_:)` **[NEW]** — transform each value into a publisher
- `Publisher.catch(_:)` **[NEW]** — recover from failure with a replacement publisher
- `Publisher.assertNoFailure(_:file:line:)` **[NEW]** — trap on failure; change failure to `Never`
- `Publisher.retry(_:)` **[NEW]** — re-subscribe on failure up to N times
- `Publisher.mapError(_:)` **[NEW]** — transform failure type
- `Publisher.replaceError(with:)` **[NEW]** — substitute fallback value on error
- `Publisher.debounce(for:scheduler:options:)` **[NEW]** — quiet-period gating
- `Publisher.throttle(for:scheduler:latest:)` **[NEW]** — rate limiting
- `Publisher.delay(for:scheduler:options:)` **[NEW]** — deferred delivery
- `Publisher.receive(on:options:)` **[NEW]** — downstream scheduler
- `Publisher.subscribe(on:options:)` **[NEW]** — upstream scheduler
- `Publisher.removeDuplicates()` / `removeDuplicates(by:)` **[NEW]**
- `Publisher.filter(_:)` **[NEW]**
- `Publisher.compactMap(_:)` **[NEW]**
- `Publisher.collect()` / `collect(_:)` **[NEW]** — buffer values
- `Publisher.combineLatest(_:)` / `combineLatest(_:_:)` **[NEW]**
- `Publisher.merge(with:)` **[NEW]**
- `Publisher.zip(_:)` **[NEW]**
- `Publisher.share()` **[NEW]** — multicast via PassthroughSubject
- `Publisher.eraseToAnyPublisher()` **[NEW]**
- `Publisher.sink(receiveCompletion:receiveValue:)` **[NEW]** — returns `AnyCancellable`
- `Publisher.assign(to:on:)` **[NEW]** — key-path assignment; returns `AnyCancellable`
- `@Published` property wrapper **[NEW]** — adds publisher to stored property; access via `$`

### Foundation (updated)
- `NotificationCenter.Publisher` **[NEW]** — `NotificationCenter.default.publisher(for:object:)`
- `RunLoop` and `DispatchQueue` now conform to `Scheduler` **[NEW]**
- `Timer.publish(every:tolerance:on:in:options:)` **[NEW]** — repeating timer publisher
- `URLSession.DataTaskPublisher` **[NEW]** — async data task as a publisher

### SwiftUI (referenced)
- `BindableObject` protocol (WWDC19) — `didChange: Publisher` property; predecessor to `ObservableObject`
- `@ObjectBinding` (WWDC19) — view property for binding to a `BindableObject`

## Code Highlights

`flatMap` + nested `catch` for per-element error recovery:
```swift
NotificationCenter.default
    .publisher(for: .newMagicTrick)
    .compactMap { $0.userInfo?["data"] as? Data }
    .flatMap { data in
        Just(data)
            .decode(type: MagicTrick.self, decoder: JSONDecoder())
            .catch { _ in Just(MagicTrick.placeholder) }
    }
    .map(\.name)
    .receive(on: RunLoop.main)
    .assign(to: \.trickNameLabel.text, on: self)
    .store(in: &cancellables)
```

`@Published` + `CombineLatest` for form validation:
```swift
@Published var password = ""
@Published var passwordAgain = ""

var validatedPassword: AnyPublisher<String?, Never> {
    $password.combineLatest($passwordAgain) { password, repeat in
        guard password == repeat, password.count >= 8 else { return nil }
        return password
    }
    .map { $0 == "password" ? nil : $0 }   // reject bad passwords
    .eraseToAnyPublisher()
}
```

`Future` wrapping a callback API:
```swift
func isUsernameAvailable(_ username: String) -> Future<Bool, Never> {
    Future { promise in
        checkUsernameOnServer(username) { available in
            promise(.success(available))
        }
    }
}

var validatedUsername: AnyPublisher<String?, Never> {
    $username
        .debounce(for: 0.5, scheduler: RunLoop.main)
        .removeDuplicates()
        .flatMap { name in
            self.isUsernameAvailable(name)
                .map { $0 ? name : nil }
        }
        .eraseToAnyPublisher()
}
```

Wiring combined validation to a UIButton:
```swift
validatedUsername
    .combineLatest(validatedPassword) { username, password -> Credentials? in
        guard let u = username, let p = password else { return nil }
        return Credentials(username: u, password: p)
    }
    .map { $0 != nil }
    .receive(on: RunLoop.main)
    .assign(to: \.isEnabled, on: signUpButton)
    .store(in: &cancellables)
```

## Takeaways
- Use `flatMap` with a nested `catch` (not a top-level `catch`) when you need per-element error recovery in an ongoing stream — a top-level `catch` terminates the outer upstream, while nested `catch` keeps it alive.
- `@Published` + `CombineLatest` is the idiomatic pattern for combining multiple form fields into a single derived validation publisher with no imperative glue code.
- Always `debounce` + `removeDuplicates` before any network call driven by user text input — these two operators together prevent server spam from rapid typing and avoid redundant calls when the text returns to the same value.
- `eraseToAnyPublisher()` should be called at every API boundary; it decouples consumers from internal operator chain types and makes it possible to refactor the chain without breaking callers.
- Store every `AnyCancellable` in a property or `Set`; forgetting to store it causes the subscription to cancel immediately on the next run loop tick.

---
_Source: WWDC19 Session 721 page (abstract and full transcript)._
