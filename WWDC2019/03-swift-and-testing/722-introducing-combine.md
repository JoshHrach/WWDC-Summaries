# Introducing Combine
**WWDC19 · Session 722** · [Watch](https://developer.apple.com/videos/play/wwdc2019/722/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, tvOS 13, watchOS 6

## Overview
Combine is Apple's new unified declarative framework for processing values over time, introduced at WWDC19. It provides a composable, type-safe, request-driven approach to handling asynchronous events — from network requests and key-value observation to notifications and UI callbacks — without replacing existing APIs. Instead, Combine adapts those APIs into a common model of Publishers, Subscribers, and Operators.

Written entirely in Swift and leveraging generics, Combine allows developers to write generic algorithms about asynchronous behavior once and apply them to all kinds of asynchronous sources. Its design emphasizes composition first: individual operators are small and focused, doing one thing each, and their power emerges from chaining them together into declarative, readable pipelines.

The session uses a student sign-up form to illustrate the problem: combining a debounced network validation request, a local password-matching check, and a UI update requires coordinating Target/Action, timers, KVO, completion blocks, and URLSession — all with mismatched types. Combine handles this with a single readable chain.

## Key Topics

**Three Core Concepts**
The entire framework rests on three abstractions:
- **Publisher**: describes how values and errors are produced (value type / struct). Has associated types `Output` and `Failure`. Allows registration of a `Subscriber`.
- **Subscriber**: receives values and a completion (reference type / class). Has `Input` and `Failure` associated types. Controls data flow via a `Subscription`.
- **Operator**: adopts `Publisher`, transforms values from an upstream Publisher and forwards them downstream. Also a value type.

**Declarative Operator API**
Operators follow the same naming conventions as Swift Collection APIs to aid discoverability:
- Functional transforms: `map`, `filter`, `reduce`, `compactMap`
- List operations: `prefix`, `first`, `dropFirst`
- Error handling: `replaceError(with:)`, `catch`
- Thread/Queue movement: `receive(on:)`, `subscribe(on:)`
- Scheduling and time: RunLoop and DispatchQueue integration, `timer`, `timeout`, `debounce`, `throttle`
- Decoding: `decode(type:decoder:)`

**Zip**
Converts multiple upstream inputs into a single tuple, requiring a value from every upstream before emitting — a "when A and B and C have all completed" pattern. Useful for gating actions on parallel async operations.

**CombineLatest**
Converts multiple upstream inputs into a single computed value, emitting whenever any upstream changes. Stores the last value from each upstream and runs a closure to combine them — a "when any of these changes" pattern. Useful for reactive form validation.

**Backpressure / Request-Driven Design**
Subscribers request values from Publishers via a `Subscription`, enabling precise control of memory and performance rather than having data pushed unbounded.

**Cancelation**
Operators like `prefix(_:)` automatically cancel upstreams when a limit is reached. `Cancellable` is returned by `Assign` and `sink` for manual teardown.

**Incremental Adoption**
Combine is designed to be adopted piece-by-piece. Existing `NotificationCenter`, `URLSession`, and KVO APIs can be wrapped as Publishers; no full rewrite required.

## APIs & Frameworks

**Combine** **[NEW]**

Core protocols:
- `Publisher` protocol — `Output`, `Failure` associated types; `subscribe(_:)` **[NEW]**
- `Subscriber` protocol — `Input`, `Failure`; `receive(subscription:)`, `receive(_:)`, `receive(completion:)` **[NEW]**
- `Subscription` protocol — `request(_:)`, `cancel()` **[NEW]**
- `Cancellable` protocol **[NEW]**
- `AnyCancellable` **[NEW]**

Built-in Publishers:
- `NotificationCenter.Publisher` **[NEW]**
- `URLSession.DataTaskPublisher` **[NEW]**
- `Timer.TimerPublisher` **[NEW]**
- `Future` — single async value **[NEW]**
- `Just` — single synchronous value **[NEW]**
- `PassthroughSubject` **[NEW]**
- `CurrentValueSubject` **[NEW]**

Built-in Subscribers:
- `Subscribers.Assign` — writes received values to a key path on an object **[NEW]**
- `Subscribers.Sink` — closure-based subscriber **[NEW]**

Operators (selected):
- `map(_:)` **[NEW]**
- `compactMap(_:)` **[NEW]**
- `filter(_:)` **[NEW]**
- `reduce(_:_:)` **[NEW]**
- `prefix(_:)` — cancels upstream after N values **[NEW]**
- `first()`, `dropFirst(_:)` **[NEW]**
- `zip(_:)` / `zip(_:_:)` / `zip(_:_:_:)` **[NEW]**
- `combineLatest(_:)` / `combineLatest(_:_:)` / `combineLatest(_:_:_:)` **[NEW]**
- `replaceError(with:)` **[NEW]**
- `catch(_:)` **[NEW]**
- `decode(type:decoder:)` **[NEW]**
- `receive(on:)` — schedule delivery on a scheduler (RunLoop, DispatchQueue) **[NEW]**
- `subscribe(on:)` **[NEW]**
- `debounce(for:scheduler:)` **[NEW]**
- `throttle(for:scheduler:latest:)` **[NEW]**
- `timeout(_:scheduler:)` **[NEW]**
- `flatMap(maxPublishers:_:)` **[NEW]**

**Foundation Additions**
- `NotificationCenter.publisher(for:object:)` **[NEW]**
- `URLSession.dataTaskPublisher(for:)` **[NEW]**
- `Timer.publish(every:on:in:)` **[NEW]**
- KVO publisher: `NSObject.publisher(for:)` **[NEW]**

**Schedulers**
- `RunLoop` conformance to `Scheduler` **[NEW]**
- `DispatchQueue` conformance to `Scheduler` **[NEW]**
- `OperationQueue` conformance to `Scheduler` **[NEW]**

## Code Highlights

Basic Publisher → Operator → Subscriber chain:
```swift
NotificationCenter.default
    .publisher(for: .graduated, object: merlin)
    .map { note in note.userInfo?["NewGrade"] as? Int ?? 0 }
    .assign(to: \.grade, on: merlin)
```

Using `compactMap` and `filter` to sanitize values:
```swift
NotificationCenter.default
    .publisher(for: .graduated, object: merlin)
    .compactMap { note in note.userInfo?["NewGrade"] as? Int }
    .filter { grade in grade >= 5 }
    .prefix(3)
    .assign(to: \.grade, on: merlin)
```

Using `zip` to gate on three parallel async operations:
```swift
Publishers.Zip3(wandPublisher, cloakPublisher, booksPublisher)
    .map { wand, cloak, books in wand && cloak && books }
    .assign(to: \.isEnabled, on: continueButton)
```

Using `combineLatest` for reactive form validation:
```swift
Publishers.CombineLatest3(termsSwitch1, termsSwitch2, termsSwitch3)
    .map { t1, t2, t3 in t1 && t2 && t3 }
    .assign(to: \.isEnabled, on: playButton)
```

## Takeaways
- Combine provides a single, composable abstraction layer over all of Cocoa's asynchronous patterns — notifications, KVO, URLSession, timers, and closures — without replacing any of them.
- The Publisher/Subscriber/Operator model is type-safe end-to-end, catching type mismatches at compile time rather than runtime.
- Operator names intentionally mirror Swift Collection APIs (`map`, `filter`, `compactMap`, `prefix`, `zip`, `reduce`), making discovery intuitive for Swift developers.
- Adopt incrementally: wrap one asynchronous boundary at a time; see the companion session "Combine in Practice" for error handling, scheduling, and SwiftUI integration.

---
_Source: WWDC19 Session 722 page (abstract, chapter summaries, code samples, and resource links)._
