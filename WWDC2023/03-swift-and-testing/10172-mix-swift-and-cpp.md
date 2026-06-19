# Mix Swift and C++
**WWDC23 · Session 10172** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10172/)

_Platforms:_ iOS, macOS, tvOS, watchOS, visionOS, Linux, Windows (Swift 5.9 / Xcode 15)

## Overview
Swift and C++ interoperability, new in Xcode 15 and Swift 5.9, enables truly bidirectional calling between Swift and C++ without writing an Objective-C bridging layer. C++ types and functions are imported directly as Swift APIs by the Swift compiler; Swift types and functions are exposed back to C++ via a generated header. There is no ABI overhead at call boundaries — calls between the two languages are direct and native.

The session demonstrates adding a SwiftUI photo picker to a photo-editing app whose core image-processing framework is written in C++, showing both directions of interop and how to use C++ annotations to make imported APIs feel idiomatic in Swift.

## Key Topics

### Enabling C++ Interoperability
- In Xcode 15: project Build Settings → "C++ and Objective-C Interoperability" → change from `C and Objective-C` to `C++ and Objective-C++`
- No bridging header needed when importing a C++ framework; Swift compiler imports C++ module APIs automatically
- Available on: iOS, macOS, tvOS, watchOS, visionOS, Linux, Windows

### Calling C++ from Swift
- Import the C++ module with `import ModuleName` — identical to importing a Swift module
- C++ class members, free functions, and operators are immediately callable as Swift APIs
- Command-click on the module name in Xcode to inspect the Swift representation of C++ APIs
- C++ types are imported by default as Swift value types (structs) using copy constructors for lifetime management
- C++ `std::vector` and other STL containers with `begin()`/`end()` are automatically imported as Swift `Collection` — enables `for-in`, `map`, `filter`, etc.
- Unsafe C++ iterator APIs are marked unavailable by the compiler with suggestions to use safe Swift Collection APIs instead

### Calling Swift from C++
- Make Swift types and functions `public` to expose them
- Import the generated header `#import "ModuleName-Swift.h"` in C++/Objective-C++ files
- Construct and use Swift structs, call Swift methods — code completion and jump-to-definition work across languages
- Swift generic types (Array), resilient types, structs, classes, and their methods can all be exposed to C++

### C++ Annotations (from `swift/bridging`)
Add `#import <swift/bridging>` to C++ headers to unlock annotation macros:

**`SWIFT_SHARED_REFERENCE(retain, release)`**
- Marks a C++ type as having reference semantics; imported as a Swift class instead of struct
- Swift automatically performs ARC reference counting using the specified retain/release functions
- Eliminates `UnsafePointer<T>` in the Swift-facing API; type appears directly without pointer indirection
- The retain and release functions must be provided by the C++ code

**`SWIFT_COMPUTED_PROPERTY`**
- Applied to a C++ getter (and optionally setter) to map the pair into a single Swift computed property
- Transforms `getImages()` / `setImages(_:)` into a single `images` property in Swift

**`SWIFT_RETURNS_INDEPENDENT_VALUE`**
- Corrects the compiler when it marks an API unsafe because the return value appears to depend on a pointer's lifetime; declares the return value is independently owned

**`SWIFT_NAME(name)` / `SWIFT_NAME(name(_:_:))`**
- Renames an imported function; adds argument labels for Swift-style calling convention

### C++ Value Semantics vs Reference Semantics
- C++ lacks Swift/Objective-C's clear struct-vs-class distinction; everything imports as a value type by default
- `std::vector` import copies all elements on copy (no copy-on-write); prefer Swift Collection APIs over raw vectors where possible
- Types with logical reference semantics (e.g., singleton engines, shared resources) should be annotated with `SWIFT_SHARED_REFERENCE`

### Supported C++ Constructs
- STL and custom containers with `begin()`/`end()`
- Function templates and class template specializations
- `std::shared_ptr` and similar user-defined smart pointers (Swift understands retain/release semantics)
- C++ operators mapped to equivalent Swift operators
- Free functions, member functions, static members, constructors

### Versioned Interoperability
- When Apple changes how C++ APIs are imported, a new interoperability version is created
- Developers choose when to adopt new import behavior; existing code is not broken by toolchain updates
- Feature is developed in the open Swift compiler workgroup (swift.org)

## APIs & Frameworks

- **Swift and C++ Interoperability** **[NEW in Swift 5.9 / Xcode 15]** – bidirectional Swift ↔ C++ calling
- Xcode build setting: `SWIFT_OBJC_INTEROP_MODE = objcxx` – enables C++ interop mode **[NEW]**
- `#import <swift/bridging>` – C++ header providing annotation macros **[NEW]**
- `SWIFT_SHARED_REFERENCE(retain, release)` macro **[NEW]** – maps C++ type to Swift class with ARC
- `SWIFT_COMPUTED_PROPERTY` macro **[NEW]** – maps getter/setter pair to Swift computed property
- `SWIFT_RETURNS_INDEPENDENT_VALUE` macro **[NEW]** – marks return value as independently owned
- `SWIFT_NAME(...)` macro – rename / add argument labels to C++ API in Swift
- Generated Swift header (`ModuleName-Swift.h`) – exposes Swift APIs to C++; no manual bridging layer
- `std::vector<T>` – imported as Swift `Collection`; `begin()`/`end()` → automatic `for-in` support
- `std::shared_ptr<T>` – retain/release operations understood by Swift compiler for optimization
- C++ unsafe iterator APIs – marked unavailable by compiler; safe Swift Collection APIs suggested instead
- **Swift 5.9** – language version introducing C++ interop
- **Xcode 15** – IDE with code completion, jump-to-definition, and debugger support across Swift and C++

## Code Highlights

Enable C++ interop and import the framework:
```swift
// In Xcode Build Settings: C++ and Objective-C Interoperability = C++ and Objective-C++
import CxxImageKit  // C++ framework imported directly — no bridging header
```

Call a C++ method from Swift:
```swift
func loadImage(_ image: UIImage) {
    CxxImageEngine.shared.pointee.loadImage(image)
}
```

Iterate over a C++ `std::vector` using Swift Collection APIs:
```swift
for image in CxxImageEngine.shared.pointee.getImages() {
    let uiImage = CxxImageEngine.shared.pointee.uiImageFrom(image)
    UIImageWriteToSavedPhotosAlbum(uiImage, nil, nil, nil)
}
```

Call Swift from Objective-C++:
```objc
#import "SampleApp-Swift.h"

- (IBAction)openPhotoLibrary:(UIButton *)sender {
    SampleApp::ImagePicker::init().present(self);
}
```

Annotate a C++ type as a reference type (maps to Swift class with ARC):
```cpp
#import <swift/bridging>

struct SWIFT_SHARED_REFERENCE(IKRetain, IKRelease) CxxImageEngine {
    // ...
};
```

Map a C++ getter to a Swift computed property:
```cpp
SWIFT_COMPUTED_PROPERTY
inline std::vector<Image *_Nonnull> getImages() const;
// Swift sees: var images: std::vector<Image> { get }
```

Mark a return value as independently owned:
```cpp
SWIFT_RETURNS_INDEPENDENT_VALUE
std::string_view networkName() const;
```

## Takeaways
- C++ interop in Swift 5.9/Xcode 15 requires a single build setting change; after that, C++ module APIs are imported automatically with no bridging header and no call overhead.
- Use `SWIFT_SHARED_REFERENCE` for any C++ type that has reference semantics (singletons, shared resources); without it the type appears as `UnsafePointer<T>` in Swift and is easy to misuse.
- C++ STL containers are automatically usable as Swift Collections via `for-in`, `map`, and `filter`; prefer these safe APIs over raw C++ iterators which are marked unavailable by the Swift compiler.
- The feature is versioned — adopting it today is safe; Apple will introduce a new version for breaking import behavior changes rather than silently breaking existing code.

---
_Source: WWDC23 Session 10172 page (abstract, chapter summaries, code samples, and transcript)._
