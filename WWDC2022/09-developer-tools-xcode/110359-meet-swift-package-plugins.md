# Meet Swift Package plugins
**WWDC22 · Session 110359** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110359/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
Swift Package plugins, introduced in Xcode 14 and Swift Package Manager 5.6, allow Swift packages to extend the developer workflow and Xcode build system with custom actions written in Swift. A plugin is a Swift script that runs as a separate sandboxed process and interacts with Xcode or the package manager through a dedicated `PackagePlugin` API. Plugins can be private to a package or exposed as package products for use by other packages, analogous to libraries but without adding runtime content to apps.

There are two kinds of plugins: **command plugins**, which run on demand and can modify source files with user permission, and **build tool plugins**, which extend the build system's dependency graph to generate source code or resources at build time. Both types eliminate ad hoc shell scripts by providing a structured, discoverable extensibility model.

The session walks through invoking an existing command plugin from the Xcode context menu and from the `swift package plugin` command-line interface, then demonstrates configuring a build tool plugin to auto-generate Swift wrappers from data files, removing the need to check generated source into the repository.

## Key Topics

### Command Plugins
- Invoked on demand via Xcode's context menu on a package or via `swift package plugin <verb>` in Terminal.
- Can accept custom arguments and can request write permission to modify source files.
- User is warned and asked to approve any plugin that needs filesystem write access; CI workflows can bypass prompts with a flag.
- Example use cases: source code formatting (SwiftFormat integration), contributor list generation from Git history, copyright date updates.
- Plugin conforms to `CommandPlugin` protocol and implements `performCommand(context:arguments:)`.

### Build Tool Plugins
- Applied per-target via a `plugins` parameter in the package manifest (SwiftPM 5.6+).
- Do not execute immediately; instead they return `Command` values that Xcode runs as part of the build.
- Two command kinds: ordinary **build commands** (run when outputs are missing or inputs changed) and **prebuild commands** (run before every build, for cases where output names are unknown ahead of time).
- Outputs are written to the build directory, keeping the repository clean.
- Plugin conforms to `BuildToolPlugin` and implements `createBuildCommands(context:target:)`.

### Xcode Project Support
- Via `XcodeProjectPlugin` module (imported conditionally), plugins can also operate on Xcode projects, not just Swift packages.
- Separate entry-point protocols: `XcodeCommandPlugin` / `XcodeBuildToolPlugin`.

### Plugin Sandboxing
- Plugins cannot access the network or write to arbitrary filesystem locations.
- Command plugins that need source-directory write access must declare this in their manifest; Xcode prompts the user.
- Build tool plugin commands run in a sandbox preventing network access and package modification.

## APIs & Frameworks

**PackagePlugin module** **[NEW]**
- `CommandPlugin` protocol — **[NEW]** entry point for on-demand command plugins
  - `func performCommand(context: PluginContext, arguments: [String]) throws`
- `BuildToolPlugin` protocol — **[NEW]** entry point for build tool plugins
  - `func createBuildCommands(context: PluginContext, target: Target) throws -> [Command]`
- `PluginContext` — **[NEW]** provides access to the input package, its source files, and dependency information
- `Target` — representation of a package target passed to build tool plugins
- `Command` — **[NEW]** describes a tool invocation (command line, inputs, outputs) returned by a build tool plugin
  - Ordinary build command (input/output paths)
  - Prebuild command (runs before every build)

**XcodeProjectPlugin module** **[NEW]** (conditional import)
- `XcodeCommandPlugin` protocol — **[NEW]**
  - `func performCommand(context: XcodePluginContext, arguments: [String]) throws`
- `XcodeBuildToolPlugin` protocol — **[NEW]**
  - `func createBuildCommands(context: XcodePluginContext, target: XcodeTarget) throws -> [Command]`
- `XcodePluginContext` — **[NEW]**
- `XcodeTarget` — **[NEW]**

**Package Manifest (PackageDescription)**
- `plugins` parameter on `Target` — **[NEW]** list of `PluginUsage` values specifying build tool plugins for a target
- Plugin product type in `Package.swift` for exporting plugins to other packages

**Swift Package Manager CLI**
- `swift package plugin --list` — **[NEW]** lists available plugin commands
- `swift package <verb>` — **[NEW]** invokes a command plugin by its declared verb

## Code Highlights

General plugin structure with optional Xcode project support:
```swift
import PackagePlugin

@main
struct MyPlugin: CommandPlugin {
    func performCommand(context: PluginContext, arguments: [String]) throws {
        debugPrint(context)
    }
}

#if canImport(XcodeProjectPlugin)
import XcodeProjectPlugin

extension MyPlugin: XcodeCommandPlugin {
    func performCommand(context: XcodePluginContext, arguments: [String]) throws {
        debugPrint(context)
    }
}
#endif
```

Build tool plugin structure:
```swift
import PackagePlugin

@main
struct MyPlugin: BuildToolPlugin {
    func createBuildCommands(context: PluginContext, target: Target) throws -> [Command] {
        debugPrint(context)
        return []
    }
}
```

## Takeaways
- Swift package plugins replace ad hoc scripts with a structured, versioned, and sandboxed extensibility model available in both Xcode and the command line.
- Command plugins run on demand and must request permission to modify source files, giving users visibility and control.
- Build tool plugins integrate with the build system to generate code or resources automatically, keeping generated files out of the repository.
- The `XcodeProjectPlugin` module lets a single plugin support both Swift packages and native Xcode projects via conditional compilation.

---
_Source: WWDC22 Session 110359 page (abstract, chapter summaries, code samples, and resource links)._
