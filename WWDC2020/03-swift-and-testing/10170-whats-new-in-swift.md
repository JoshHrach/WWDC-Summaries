# What's New in Swift
**WWDC20 · Session 10170** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10170/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14, watchOS 7, Linux (Ubuntu, CentOS, Amazon Linux 2), Windows (Swift 5.3)

## Overview
Swift 5.3 delivered meaningful improvements across runtime performance, developer experience, and the language itself. Binary size for SwiftUI apps was cut by over 40%, heap memory overhead was reduced to less than a third of Swift 5.2 levels, and the Standard Library moved below Foundation in the stack to enable use in low-level daemons and frameworks. On the language side, over a dozen new features shipped across Swift 5.2 and 5.3, including multiple trailing closures, KeyPath as function, `@main` entry points, improved implicit `self` in closures, and synthesized `Comparable` conformance for enums.

The session also highlighted a growing set of open-source Swift packages — Swift Numerics, Swift Argument Parser, Apple Archive, Swift System, and the Swift Standard Library Preview package — as first-class additions to the developer toolkit.

## Key Topics

### Runtime Performance
- **Code size** — SwiftUI app code size reduced by over 40% in Swift 5.3; overall Swift apps approach 1.4x (not 2.3x) of equivalent Objective-C code size
- **Heap memory** — Runtime caches and protocol conformance data were drastically reduced; Swift 5.3 uses less than one-third the heap memory of Swift 5.2 (full benefit requires iOS 14 deployment target)
- **Standard Library stack position** — Moved below Foundation, enabling use in C-level frameworks and OS daemons

### Compiler & Developer Experience
- **Diagnostics** — New diagnostic strategy produces precise, actionable errors pointing to the exact source location; more additional notes guide fix-its
- **Code completion** — Up to 15x speed improvement vs. Xcode 11.5; works correctly in ternary expressions and with KeyPath-as-function
- **Code indentation** — Improved alignment for chained method calls, call arguments, tuple elements, multi-line collection elements (critical for SwiftUI code)
- **Debugger** — Displays human-readable reasons for Swift runtime failure traps; LLDB falls back to DWARF debug info when Clang module import fails, improving reliability of variable view and expression evaluator

### Language Features (Swift 5.2 & 5.3)
- **Multiple trailing closure syntax** — All closure arguments can use trailing closure syntax, not just the final one
- **KeyPath expressions as functions** — A KeyPath argument can be passed to any function parameter with a matching signature (Swift 5.2)
- **`@main`** — Generalized type-based program entry point attribute; library authors declare `static main()` on a protocol/superclass; users tag their type with `@main`
- **Implicit `self` in closures** — Including `self` in the capture list allows omitting it from the closure body; if `self` is a struct/enum, it can be omitted entirely
- **Multi-pattern catch clauses** — `catch` clauses now support the full power of `switch` case patterns
- **Synthesized `Comparable` for enums** — Compiler synthesizes `Comparable` conformance for qualifying enum types (no associated values required)
- **Enum cases as protocol witnesses** — Enum cases can fulfill `static var` and `static func` protocol requirements
- **Result builders / embedded DSLs** — Extended to support `if-let` and `switch` pattern matching; builder attribute can be inferred from protocol requirement (builder inference)
- **`Float16`** — New IEEE 754 half-precision floating-point type; 2 bytes; doubles SIMD throughput on supported hardware

### New SDK Libraries
- **Apple Archive** — Modular archive format for fast multithreaded compression; Swift API; Finder and command-line integration
- **Swift System** — Strongly-typed, idiomatic Swift wrappers over POSIX/Darwin system calls; `FilePath` and related currency types
- **OSLog enhancements** — String interpolation support with formatting and privacy options; compiler-optimized for minimal overhead

### Open-Source Swift Packages
- **Swift Numerics** — Basic math functions (`sin`, `log`, etc.) usable in generic contexts; complex number support layout-compatible with C
- **Swift Argument Parser** — Declarative command-line argument parsing with automatic help generation and validation
- **Swift Standard Library Preview** — Early access to approved Swift Evolution proposals before official release; seeded with SE-0270 (sub-range operations and `RangeSet`)
- **Swift AWS Lambda Runtime** — Open-source serverless runtime for AWS Lambda

### Platform Expansion
- Official support: Ubuntu (updated), CentOS 8, Amazon Linux 2 (new), Windows (initial, Swift 5.3)
- Swift is used for AWS Lambda serverless functions via the open-source Swift AWS Lambda Runtime

## APIs & Frameworks

### Swift Language
- Multiple trailing closure syntax **[NEW — SE-0279]**
- `KeyPath` as function **[NEW — SE-0249, Swift 5.2]**
- `@main` attribute **[NEW — SE-0281]**
- Implicit `self` capture list shorthand **[NEW — SE-0269]**
- Struct/enum: omit `self` entirely in closures **[NEW — SE-0269]**
- Multi-pattern catch clauses **[NEW — SE-0276]**
- Synthesized `Comparable` for enums **[NEW — SE-0266]**
- Enum cases as protocol witnesses **[NEW — SE-0280]**
- Result builder `if-let` and `switch` support **[NEW — SE-0289]**
- Result builder inference **[NEW — SE-0289]**
- `Float16` type **[NEW — SE-0277]**

### SDK
- `Logger` / `OSLog` — `log(_:)` with string interpolation, alignment, formatting, privacy options **[UPDATED]**
- `AppleArchive` framework **[NEW]** — `ArchiveByteStream`, `ArchiveStream`, `ArchiveStream.withEncodeStream`
- Swift System — `FilePath`, `FileDescriptor`, strongly-typed system call wrappers **[NEW]**

### Open-Source Packages (Swift Package Manager)
- `swift-numerics` — `Real` protocol, complex number arithmetic **[NEW]**
- `swift-argument-parser` — `ParsableCommand`, `@Argument`, `@Option`, `@Flag` **[NEW]**
- `swift-standard-library-preview` — `RangeSet`, collection sub-range operations **[NEW]**
- `swift-aws-lambda-runtime` — `Lambda.run(_:)` **[NEW]**

## Code Highlights

`@main` entry point with ArgumentParser:
```swift
import ArgumentParser

@main
struct Hello: ParsableCommand {
    @Argument(help: "The name to greet.")
    var name: String
    func run() { print("Hello, \(name)!") }
}
```

Synthesized `Comparable` for enum:
```swift
enum MessageStatus: Hashable, Comparable {
    case draft, saved, failedToSend, sent, delivered, read
    var wasSent: Bool { self >= .sent }
}
```

Apple Archive compression:
```swift
import AppleArchive
try ArchiveByteStream.withFileStream(path: "/tmp/Photos.aar", mode: .writeOnly, ...) { file in
    try ArchiveByteStream.withCompressionStream(using: .lzfse, writingTo: file) { compressor in
        try ArchiveStream.withEncodeStream(writingTo: compressor) { encoder in
            try encoder.writeDirectoryContents(archiveFrom: source, keySet: fieldKeySet)
        }
    }
}
```

OSLog string interpolation with formatting and privacy:
```swift
logger.log("\(offerID, align: .left(columns: 10), privacy: .public)")
logger.log("\(seconds, format: .fixed(precision: 2)) seconds")
```

## Takeaways

- Swift 5.3 cuts SwiftUI app binary size by over 40% and heap memory to less than a third of Swift 5.2, with full benefits at iOS 14 deployment target.
- Multiple trailing closure syntax and `@main` are the headline language additions, improving SwiftUI API ergonomics and standardizing executable entry points.
- The Standard Library moving below Foundation is architecturally significant — it opens the door for Swift in Apple's lowest-level system components.
- Apple Archive, Swift System, Swift Numerics, and Swift Argument Parser bring production-quality open-source Swift APIs to standard developer workflows via Swift Package Manager.

---
_Source: WWDC20 Session 10170 page (abstract, transcript, code samples, and resource links)._
