# Go Small with Embedded Swift
**WWDC24 · Session 10197** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10197/)

_Platforms:_ Embedded devices (ARM, RISC-V); Apple platforms via Secure Enclave Processor

## Overview
Embedded Swift is a new compilation mode that brings the safety and expressivity of Swift to constrained environments such as microcontrollers. This session introduces the concept, demonstrates building a HomeKit-compatible smart LED light using an ESP32-C6 board (RISC-V), and explains how Embedded Swift differs from full Swift by forming a strict subset with no runtime, no reflection, and no existential types.

The demo walks through progressively building a Matter protocol-based accessory: starting with a bare-metal "Hello, World!" entry point, adding Swift wrappers over C APIs for LED control, and integrating the Matter C++ SDK to create a fully HomeKit-controllable device controllable from the Home app.

## Key Topics

**Why Embedded Swift**
- C and C++ have dominated embedded development; Embedded Swift brings Swift's ergonomics, type safety, and expressive APIs
- Used inside Apple on the Secure Enclave Processor; memory safety is critical in low-level, sensitive code
- Currently an experimental feature; requires preview toolchains from swift.org
- Supports ARM (32/64-bit) and RISC-V (32/64-bit); hardware-agnostic design makes porting straightforward

**Getting Started**
- Uses vendor-provided C SDKs via Swift's C interoperability and a bridging header
- CMake integration; LSP support for autocompletion and real-time error checking in editors like Neovim
- Entry point defined with `@_cdecl("app_main")` — C-callable Swift function required by the vendor SDK
- `print()` works out of the box for device logging

**Swift Abstractions over C APIs**
- Swift's C interop allows calling C functions directly; building a wrapper type (e.g., `LED` struct) provides ergonomic, type-safe APIs
- Properties like `led.enabled: Bool`, `led.brightness: Int`, `led.color` replace raw C function calls
- Swift enums with associated values replace C unions/callbacks — e.g., a `Color` enum with `.hueSaturation(Int, Int)` and `.temperature(Int)` cases

**Matter Protocol Integration**
- Matter (open standard for smart home) is implemented via C++ SDK; Swift's C++ interoperability enables direct use
- Wrapper types in Swift (`Matter.Node`, `Matter.ExtendedColorLight`, `Matter.Application`) provide clean API surface
- Closures serve as ergonomic callbacks replacing C function pointers and void context pointers
- `lightEndpoint.eventHandler` closure receives typed Swift enum values for attribute updates (`.onOff`, `.levelControl`, `.colorControl(.currentHue)`, etc.)

**How Embedded Swift Differs from Full Swift**
- Embedded Swift is strictly a subset — all Embedded Swift code also runs on full Swift with identical behavior
- **Disallowed features**: runtime reflection (`Mirror` APIs), metatypes, `any` existential types
- **Fully supported**: value types, reference types, closures, optionals, error handling, generics (via specialization), enums with associated values, pattern matching
- When unavailable features are used, the compiler produces a clear error
- Replace `any Protocol` with `some Protocol` (generics) to avoid existential overhead

## APIs & Frameworks

**Embedded Swift Language**
- `@_cdecl("symbol_name")` — expose a Swift function as a C-callable symbol **[NEW/EXPERIMENTAL]**
- Embedded Swift compilation mode (`-experimental-feature Embedded`) — **[NEW/EXPERIMENTAL]**
- Full generics support via specialization (no existential overhead)
- Swift closures as callbacks (replaces C function pointers + void context)
- Swift enums with associated values for typed event dispatching
- `Int.random(in:)`, `sleep(_:)` — available in Embedded Swift

**Swift C++ Interoperability**
- Bridging header to import vendor C/C++ SDK APIs
- Direct calls to C functions from Swift (e.g., `led_driver_set_hue`, `led_driver_init`)
- C++ types and APIs accessible via Swift's C++ interop

**External Libraries and Resources**
- **Swift MMIO** (`apple/swift-mmio`) — safe, structured APIs for memory-mapped hardware registers
- **Swift Embedded Example Projects** (`apple/swift-embedded-examples`) — reference projects for popular boards (ARM, RISC-V, Playdate)
- **Swift Matter Examples** (`apple/swift-matter-examples`) — this demo's source code on GitHub
- Swift Forums Embedded subcategory — community discussion and support

## Code Highlights

Minimal Embedded Swift entry point:
```swift
@_cdecl("app_main")
func app_main() {
    print("Hello, Embedded Swift!")
}
```

Ergonomic LED control using Swift wrapper:
```swift
led.color = .red
led.brightness = 80
while true {
    sleep(1)
    led.enabled = !led.enabled
    if led.enabled {
        led.color = .hueSaturation(Int.random(in: 0..<360), 100)
    }
}
```

Replacing `any` types with generics (required in Embedded Swift):
```swift
// Not allowed in Embedded Swift:
func count(countable: any Countable) { ... }
// Correct approach:
func count(countable: some Countable) { ... }
```

## Takeaways
- Embedded Swift is a strict subset of full Swift — write code in Embedded Swift and it compiles and runs on full Swift too.
- Use `@_cdecl` to expose your Swift entry point to vendor C build systems, then build ergonomic Swift wrappers over the C APIs.
- Avoid `any` existential types and `Mirror` reflection; use generics and concrete types instead — the compiler will tell you when you cross a boundary.
- Explore the `apple/swift-embedded-examples` and `apple/swift-matter-examples` repos as starting templates.

---
_Source: WWDC24 Session 10197 page (abstract, chapter summaries, code samples, and resource links)._
