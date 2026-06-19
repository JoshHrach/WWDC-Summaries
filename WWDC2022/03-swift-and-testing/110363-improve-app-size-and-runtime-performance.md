# Improve App Size and Runtime Performance
**WWDC22 · Session 110363** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110363/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
This session covers four complementary optimizations to the Swift and Objective-C runtimes and compilers in Xcode 14 that reduce binary size and improve launch-time and runtime performance. Uniquely, none of these improvements require code changes — they are transparent to developers and activate automatically based on the Xcode version used to build and/or the OS deployment target set.

The improvements span two layers: the compiler/linker (Xcode 14, effective for all deployment targets) and the OS runtime (effective when the minimum deployment target is iOS 16 / tvOS 16 / watchOS 9). Some improvements even apply retroactively to existing app binaries simply by running on the new OS.

## Key Topics
- **Swift protocol check precomputation** — Previously, Swift protocol conformance metadata was partially built at launch time (especially for generics), taking up to hundreds of milliseconds or half of total launch time on some real-world apps. The new Swift runtime precomputes this metadata as part of the dyld closure at build time. Applies automatically on iOS 16 / tvOS 16 / watchOS 9, even for existing app binaries.
- **Smaller Objective-C message send stubs** — `objc_msgSend` calls previously cost 12 bytes each on ARM64 (4 for the call + 8 to prepare the selector). Xcode 14 introduces selector stubs that emit selector setup code once per selector and share it, reducing per-call cost to 4 bytes — up to 8-byte savings per call site, for up to 2% binary code size reduction. Two modes: default (balance of size + performance) and `objc_stubs_small` linker flag (maximum size savings, two separate stubs).
- **Cheaper retain/release calls** — ARC-inserted `objc_retain` and `objc_release` calls previously followed the standard C calling convention, requiring extra move instructions to position the object pointer. A new specialized calling convention allows the runtime to pick the right variant based on where the pointer already lives, eliminating redundant moves — up to 4-byte savings per call (down from 8 bytes on ARM64), for up to 2% additional code size reduction. Requires deployment target iOS 16 / tvOS 16 / watchOS 9.
- **Faster and smaller autorelease elision** — The classic autorelease elision technique used a special marker instruction that the runtime read as data (expensive memory access). The new runtime uses return-address comparison instead: the autorelease records the caller's return address cheaply as a pointer, and the subsequent retain compares its return address to the saved one — eliminating the expensive memory load. The special marker instruction is also no longer needed, saving additional code bytes. Runtime speed improvement applies to existing apps on new OS; size improvement requires deployment target iOS 16 / tvOS 16 / watchOS 9.

## APIs & Frameworks
These optimizations involve no new public APIs. The relevant components are:

**Swift Runtime**
- Protocol conformance metadata precomputation as part of dyld closure **[NEW behavior on iOS 16+]**
- Swift protocol check using `as` and `is` operators — now precomputed, no per-launch overhead for existing conformances

**Objective-C Runtime**
- `objc_msgSend` — reduced call-site code size via selector stubs in Xcode 14 **[NEW]**
- `objc_retain` / `objc_release` — specialized calling convention reduces code size on iOS 16+ **[NEW]**
- Autorelease elision — return-address-based comparison replaces marker instruction load; faster and smaller on iOS 16+ **[NEW]**

**Linker / Compiler (Xcode 14)**
- `-objc_stubs_small` linker flag **[NEW]** — opt into maximum code size reduction for message send stubs (at some performance cost); default is size+performance balanced mode
- Automatic ARC retain/release calling-convention optimization when targeting iOS 16 / tvOS 16 / watchOS 9

**dyld**
- dyld closure — now includes precomputed Swift protocol conformance metadata; no developer action required

## Code Highlights
No code changes are required from developers. The improvements are transparent:

- Build with **Xcode 14** to get message send stubs (up to 2% binary size reduction) regardless of deployment target.
- Set deployment target to **iOS 16 / tvOS 16 / watchOS 9** to additionally get cheaper retain/release (another ~2% size reduction) and smaller autorelease elision code.
- Run on **iOS 16 / tvOS 16 / watchOS 9** for faster Swift protocol checks and faster autorelease elision even for previously-compiled binaries.

To opt into maximum size savings for message send (at the cost of slightly more calls):
```
# Linker flag (not a source-code change)
-objc_stubs_small
```

## Takeaways
- All four improvements are transparent — no API changes, no code edits, no new build settings required from developers.
- Simply rebuilding with Xcode 14 gets up to 2% binary size savings from message send stubs.
- Updating the deployment target to iOS 16 / tvOS 16 / watchOS 9 unlocks another ~2% from cheaper retain/release and smaller autorelease elision code.
- Swift protocol check improvements apply automatically on iOS 16+ even to existing binaries, potentially cutting hundreds of milliseconds from app launch time.

---
_Source: WWDC22 Session 110363 page (abstract, chapter summaries, code samples, and resource links)._
