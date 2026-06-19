# Advancements in the Objective-C Runtime
**WWDC20 · Session 10163** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10163/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
This session dives into three low-level improvements made to the Objective-C runtime in the 2020 OS releases that significantly reduce memory usage with no developer code changes required. The changes affect class data structures, method lists, and tagged pointer representation on ARM64. While developers don't need to act on these improvements, the session warns that code directly accessing internal runtime data structures will break — and shows how to diagnose and avoid such crashes.

The core theme is that all of the runtime's internal data is accessible through stable public APIs (`class_getName`, `method_getImplementation`, `isKindOfClass:`, etc.). When apps rely on internal layouts instead of these APIs, they break whenever Apple makes the kind of improvements described here.

## Key Topics
- **`class_rw_t` split into `class_rw_ext_t`** — The read-write class data structure was halved in size by moving rarely-used fields (extended method/property lists, demangled Swift name) into a separately-allocated `class_rw_ext_t`. Only ~10% of classes ever need this extended record; system-wide savings of ~14 MB of dirty memory.
- **Clean vs. dirty memory** — Clean memory (read-only, can be evicted to disk) vs. dirty memory (must stay in RAM for the life of the process). The runtime aggressively minimizes dirty memory; iOS has no swap, making dirty memory especially costly.
- **Relative method lists** — Method table entries previously used 64-bit absolute pointers (24 bytes per method entry). Now they use 32-bit relative offsets within the binary (12 bytes per entry). Benefits: no linker fixups at load time, entries can be stored in true read-only memory (more secure), and ~40 MB saved system-wide on a typical iPhone.
- **Swizzling with relative method lists** — A global table maps swizzled methods to their new implementations; swizzling a method dirties only a small table entry instead of an entire memory page.
- **Deployment target and relative method lists** — Building with a minimum deployment target of iOS 14 / macOS Big Sur causes Xcode to emit relative method lists. Targeting older OSes produces old-style pointer-based lists (still fully supported).
- **Crash recognition for wrong deployment targets** — If a binary built with relative method lists runs on an old OS, the runtime tries to interpret two 32-bit relative offsets as a single 64-bit pointer, producing a nonsense crash address that looks like two 32-bit values concatenated.
- **Tagged pointers** — Objects like small NSNumbers and NSDates can be encoded directly in a pointer-sized value without heap allocation. The tag bit distinguishes them from real pointers; extended tags (tag == 7) use 8 additional bits for type, enabling 256 more tag types.
- **ARM64 tagged pointer format change** — Previously: tag bit at bottom, tag number in next 3 bits. New format: tag bit still at top (for `objc_msgSend` optimization), tag number moves to the bottom 3 bits, extended tag uses the top byte (ARM Top Byte Ignore region). This allows tagged pointers to contain real pointers as payload, enabling references to constant data in binaries.
- **Security obfuscation** — Tagged pointer values are XOR'd with a per-process random value at startup to prevent forgery.

## APIs & Frameworks

This session covers internal runtime improvements. No new public APIs are introduced. The relevant public APIs are the stable alternatives to direct structure access:

### Objective-C Runtime (public APIs to use instead of direct structure access)
- **`class_getName(Class)`** — Returns the class name as a C string
- **`class_getSuperclass(Class)`** — Returns the superclass
- **`class_copyMethodList(Class, unsigned int *)`** — Returns the method list
- **`method_getName(Method)`** — Returns a method's selector
- **`method_getTypeEncoding(Method)`** — Returns a method's type encoding string
- **`method_getImplementation(Method)`** — Returns a method's IMP
- **`-[NSObject isKindOfClass:]`** — Type-checking that works across tagged pointer format changes
- **`CFGetTypeID` / `CFStringGetTypeID`** — CF-level type checking

### Tools
- **`heap` command-line tool** — `heap <process> | egrep 'class_rw|COUNT'` to inspect runtime data structure memory usage on macOS

## Code Highlights

Inspect `class_rw_t` memory usage for a running process:
```shell
heap Mail | egrep 'class_rw|COUNT'
```

Correct way to access class information (never read internal bits directly):
```objc
// Use public runtime APIs:
const char *name = class_getName([MyClass class]);
Class superclass = class_getSuperclass([MyClass class]);

// Type checking with tagged pointers:
if ([obj isKindOfClass:[NSString class]]) { /* safe */ }
NSUInteger length = [obj length]; /* works regardless of tagged pointer format */
```

## Takeaways
- Three runtime changes in iOS 14 / macOS Big Sur save ~14 MB (class data split) + ~40 MB (relative method lists) + additional (tagged pointer improvements) of system-wide dirty memory — all automatic, no code changes required.
- Raising your deployment target to the current OS release allows Xcode to emit relative method lists in your binaries, reducing binary size and startup memory usage.
- Never directly access internal runtime data structures (`class_rw_t`, method list entries, tagged pointer bits); any code that does so will crash on new OS versions. Always use the public Objective-C runtime APIs.
- A crash address that looks like two concatenated 32-bit values is the telltale sign of a binary built with relative method lists running on an old OS that doesn't understand them.

---
_Source: WWDC20 Session 10163 page (abstract, chapter summaries, code samples, and resource links)._
