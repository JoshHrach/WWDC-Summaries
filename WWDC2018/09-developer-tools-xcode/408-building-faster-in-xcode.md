# Building Faster in Xcode
**WWDC18 · Session 408** · [Watch](https://developer.apple.com/videos/play/wwdc2018/408/)

_Platforms:_ iOS, macOS, watchOS, tvOS (Xcode 10)

## Overview
This session presents actionable techniques to reduce both clean and incremental build times in Xcode 10. It covers two complementary strategies: increasing overall build efficiency through parallelism and correct project configuration, and reducing the amount of work performed on each incremental build through smarter source-level choices.

The project-level section focuses on target dependency graphs, Run Script phase configuration (inputs/outputs and file lists), and Xcode 10's new build timing tools. The source-level section covers Swift type-inference costs, Swift incremental compilation semantics, and the Swift–Objective-C bridging boundary — all areas where small changes can eliminate large amounts of redundant recompilation.

## Key Topics

### Parallelizing Targets
- Enable "Parallelize Build" in the scheme's Build Action options.
- Explicit target dependencies: declared in the "Target Dependencies" build phase.
- Implicit target dependencies: inferred from "Link Binary with Libraries" phase; not inferred from Autolink or `OTHER_LDFLAGS`.
- Xcode 10 new feature: target compilation can begin as soon as the dependency's build phases that satisfy compilation are complete — linking of dependencies can now overlap with compilation of dependents.
- Three dependency anti-patterns to eliminate:
  - "Do Everything" — a test target that depends on too many components; break into smaller, per-component test targets.
  - "Nosy Neighbors" — a target that depends on a sibling only for a small piece of it; extract that piece into its own target.
  - "Forgotten Ones" — dead dependencies from deleted code; remove them.

### Run Script Phases
- Input files and output files must be declared so the build system can determine whether to re-run the script and schedule it correctly.
- If no input files are declared, the script runs on every build.
- Xcode 10 new: **Input File Lists** and **Output File Lists** — external `.xcfilelist` text files listing one path per line, with access to build settings; read once at build start (cannot be generated during the build).
- Xcode 10 new: inline task durations in the build log; "Recent" filter shows tasks from the previous build only.
- Xcode 10 new: **Build with Timing Summary** (Product > Perform Action > Build with Timing Summary) — produces an aggregate timing section at the end of the build log, also accessible via `xcodebuild -showBuildTimingSummary`.
- Xcode 10 new: improved cycle detection with expanded cycle details in the build error.

### Swift Incremental Compilation (Xcode 10)
- Xcode 10 improved incremental mode to share inter-file compilation work (previously only available with Whole-Module Optimization). WMO in Debug configuration is no longer needed.
- To revert to default: in Build Settings, select the debug configuration for "Compilation Mode" and delete the override (restores to Xcode's default of incremental).
- Dependency granularity: within a module (target), dependencies are per-file; if a file's interface changes, all files that use it are recompiled. Function body changes do not invalidate other files.
- Cross-module dependencies (via `import` or bridging header) are coarse-grained — any change to the framework target triggers recompilation of all files in the importing target, unless changes are confined to function bodies.

### Complex Swift Expressions
- Explicit type annotations on stored properties (especially those initialized with complex expressions) save work in every file that uses the type.
- Avoid overly complex single-expression closures; the compiler may time out. Break into multi-statement closures.
- Avoid `AnyObject` method calls: the compiler must search all Objective-C-visible methods in the project. Use a protocol instead.
- Providing explicit types is both a performance improvement and a software engineering best practice.

### Swift–Objective-C Interface Reduction
- **Generated header** (Swift → Objective-C): mark `IBOutlet`, `IBAction`, and Notification/selector methods as `private` to remove them from the generated header; fewer declarations = fewer rebuild triggers.
- Use block/closure-based APIs instead of `#selector` to eliminate unnecessary Objective-C exposure.
- Disable "Swift 3 @objc Inference" build setting (set `SWIFT_SWIFT3_OBJC_INFERENCE` to default/off) once all explicit `@objc` annotations are in place; reduces the generated header size significantly.
- **Bridging header** (Objective-C → Swift): use Objective-C categories (nameless extension syntax in a separate `.h`/`.m` or directly in `.m`) to move seldom-used properties and their imports out of the main header. The smaller the bridging header's transitive import closure, the fewer Swift files are rebuilt when any Objective-C header changes.

## APIs & Frameworks

**Xcode Build System**
- Parallelize Build (scheme option)
- Find Implicit Dependencies (scheme option)
- Target Dependencies build phase
- Link Binary with Libraries build phase (generates implicit dependencies)
- Run Script build phase — Input Files, Output Files, Input File Lists (`.xcfilelist`), Output File Lists **[NEW in Xcode 10]**
- Build with Timing Summary (`-showBuildTimingSummary`) **[NEW in Xcode 10]**
- Inline task durations in build log **[NEW in Xcode 10]**
- Build log "Recent" filter
- Dependency cycle error with expanded details **[NEW in Xcode 10]**

**Swift Compiler**
- `SWIFT_COMPILATION_MODE` build setting — `wholemodule` vs. incremental (default)
- `SWIFT_SWIFT3_OBJC_INFERENCE` build setting — controls automatic `@objc` inference for `NSObject` subclasses
- `SWIFT_OBJC_BRIDGING_HEADER` build setting — path to bridging header
- Generated Objective-C header (`-Swift.h`)
- `*.swiftdeps` — per-file dependency graph for incremental builds
- `@objc`, `@IBOutlet`, `@IBAction` attributes
- `private` access modifier — removes declarations from generated header

**Objective-C**
- Nameless categories (`@interface MyClass ()`) — declare additional properties in `.m` or a separate internal header to shrink the bridging-header import graph

## Code Highlights

Declaring Run Script file lists (`.xcfilelist` format):
```
$(SRCROOT)/Resources/icons.png
$(SRCROOT)/Resources/logo.svg
$(DERIVED_FILE_DIR)/generated_version.swift
```

Annotating a property to avoid expensive type inference:
```swift
// Before (slow — compiler infers type from complex expression in every use site)
let bigNumber = (1...100).reduce(0, +) |> Double.init |> pow(2)

// After (fast — type is explicit)
let bigNumber: Double = (1...100).reduce(0, +) |> Double.init |> pow(2)
```

Using a protocol instead of AnyObject to limit compiler search space:
```swift
protocol ActionHandler: AnyObject {
    func handleAction()
}
// Change delegate property from:
weak var delegate: AnyObject?
// To:
weak var delegate: ActionHandler?
```

Shrinking the generated header with `private`:
```swift
class MyViewController: UIViewController {
    @IBOutlet private weak var titleLabel: UILabel!
    @IBAction private func submitTapped(_ sender: UIButton) { }
}
```

Objective-C category to keep internal property out of the bridging header:
```objc
// MyViewController+Internal.h (not imported in bridging header)
@interface MyViewController ()
@property (nonatomic, strong) MyNetworkManager *networkManager;
@end
```

## Takeaways
- "Parallelize Build" is free parallelism; always enable it and clean up dependency graph anti-patterns (Do Everything, Nosy Neighbors, Forgotten Ones).
- Always declare Run Script inputs and outputs; undeclared inputs cause the script to re-run on every build and block the new Xcode 10 parallel compilation start.
- Xcode 10's improved incremental Swift compilation replaces the WMO-in-Debug workaround; revert any manual WMO debug settings.
- Narrowing Swift–Objective-C boundaries (`private`, protocol vs. `AnyObject`, bridging-header categories) reduces both generated header size and header dependency surface, cutting down unnecessary rebuilds.

---
_Source: WWDC18 Session 408 page (abstract, full transcript, and resource links)._
