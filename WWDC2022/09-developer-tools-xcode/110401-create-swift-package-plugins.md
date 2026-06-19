# Create Swift Package Plugins
**WWDC22 · Session 110401** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110401/)

_Platforms:_ macOS Ventura 13, Xcode 14

## Overview
This session provides a hands-on walkthrough of creating all three types of Swift Package plugins introduced in Xcode 14: custom command plugins, in-build command plugins, and pre-build command plugins. The session builds three real examples — a contributor-list generator, an asset catalog constant generator, and a `genstrings`-based localization pre-processor — demonstrating how plugins automate developer workflows without leaving Xcode.

Plugins are Swift code in a `Plugins/` directory that conform to protocols from the `PackagePlugin` module. They run in a sandbox (no network, limited file system write access) and communicate back to Xcode or SwiftPM by returning command descriptions. They can be distributed as standalone packages with plugin products, making them shareable across projects.

## Key Topics

### Plugin Types
- **Custom command plugins** — invoked manually from Xcode's context menu or `swift package <verb>`; can request permission to write to the package root directory; receive user-provided arguments
- **In-build command plugins** — declared with inputs and outputs; the build system runs them only when outputs are out-of-date; used for code generation that produces a deterministic set of files
- **Pre-build command plugins** — run at the start of every build; used when outputs are not predictable; risk: expensive work runs on every build, so caching strategies are important

### Manifest Changes
- Requires `tools-version: 5.6` or later
- Plugin targets use `.plugin(name:capability:dependencies:)` in `Package.swift`
- Capabilities: `.command(intent:permissions:)`, `.buildTool()`
- Plugin products use `.plugin(name:targets:)` to expose plugins to other packages
- Executables used by build tool plugins are declared as normal `.executableTarget` dependencies

### Sandbox Model
Plugins run in a sandbox similar to the package manifest evaluator: no network access, writes restricted to the plugin's own work directory (`context.pluginWorkDirectory`). Custom commands may additionally request `.writeToPackageDirectory(reason:)` permission shown to the user at runtime.

### Plugin Context API
- `PluginContext` — provides `packageGraph`, `pluginWorkDirectory`, and `tool(named:)` for locating executables
- `PluginContext.tool(named:)` — resolves an executable dependency by name and returns its path
- `SourceModuleTarget` — target subtype with `sourceFiles(withSuffix:)` for enumerating source or resource files

### Build Commands
- `.buildCommand(displayName:executable:arguments:inputFiles:outputFiles:)` — in-build command; skipped if all outputs are newer than all inputs
- `.prebuildCommand(displayName:executable:arguments:outputFilesDirectory:)` — pre-build command; always runs; outputs appear in `outputFilesDirectory`

## APIs & Frameworks

### PackagePlugin Module
- `CommandPlugin` protocol **[NEW]** — `performCommand(context:arguments:) async throws`
- `BuildToolPlugin` protocol **[NEW]** — `createBuildCommands(context:target:) throws -> [Command]`
- `PluginContext` **[NEW]** — provides `packageGraph`, `pluginWorkDirectory: Path`, `tool(named:) throws -> PluginContext.Tool`
- `PluginContext.Tool` **[NEW]** — `.path: Path` to the resolved executable
- `Target` — base type for all targets in the package graph
- `SourceModuleTarget` **[NEW]** — target with source files; `.sourceFiles(withSuffix:) -> FileList`
- `FileList` / `File` **[NEW]** — iterate source or resource files; `File.path: Path`
- `Path` **[NEW]** — file system path type; `.string`, `.stem`, `.appending(_:)`, `.appending(subpath:)`
- `Command.buildCommand(displayName:executable:arguments:inputFiles:outputFiles:) -> Command` **[NEW]**
- `Command.prebuildCommand(displayName:executable:arguments:outputFilesDirectory:) -> Command` **[NEW]**

### Package.swift (PackageDescription)
- `.plugin(name:capability:dependencies:)` **[NEW]** — plugin target declaration
- `PluginCapability.command(intent:permissions:)` **[NEW]** — custom command capability
- `PluginCapability.buildTool()` **[NEW]** — build tool capability
- `PluginCommandIntent.custom(verb:description:)` **[NEW]** — defines SwiftPM CLI verb
- `PluginPermission.writeToPackageDirectory(reason:)` **[NEW]** — requests write access to package root
- `.plugin(name:targets:)` **[NEW]** — plugin product declaration for sharing across packages

## Code Highlights

Custom command plugin (GenerateContributors):
```swift
import PackagePlugin
import Foundation
@main
struct GenerateContributors: CommandPlugin {
    func performCommand(context: PluginContext, arguments: [String]) async throws {
        let process = Process()
        process.executableURL = URL(fileURLWithPath: "/usr/bin/git")
        process.arguments = ["log", "--pretty=format:- %an <%ae>%n"]
        let pipe = Pipe()
        process.standardOutput = pipe
        try process.run()
        process.waitUntilExit()
        let output = String(decoding: pipe.fileHandleForReading.readDataToEndOfFile(), as: UTF8.self)
        let contributors = Set(output.components(separatedBy: .newlines)).sorted().filter { !$0.isEmpty }
        try contributors.joined(separator: "\n").write(toFile: "CONTRIBUTORS.txt", atomically: true, encoding: .utf8)
    }
}
```

In-build command plugin (AssetConstants):
```swift
import PackagePlugin
@main
struct AssetConstants: BuildToolPlugin {
    func createBuildCommands(context: PluginContext, target: Target) throws -> [Command] {
        guard let target = target as? SourceModuleTarget else { return [] }
        return try target.sourceFiles(withSuffix: "xcassets").map { catalog in
            let output = context.pluginWorkDirectory.appending(["\(catalog.path.stem).swift"])
            return .buildCommand(
                displayName: "Generating constants for \(catalog.path.stem)",
                executable: try context.tool(named: "AssetConstantsExec").path,
                arguments: [catalog.path.string, output.string],
                inputFiles: [catalog.path],
                outputFiles: [output])
        }
    }
}
```

Pre-build command plugin (GenstringsPlugin):
```swift
return [.prebuildCommand(
    displayName: "Generating localized strings from source files",
    executable: .init("/usr/bin/xcrun"),
    arguments: ["genstrings", "-SwiftUI", "-o", localizationDirectoryPath] + inputFiles,
    outputFilesDirectory: localizationDirectoryPath
)]
```

## Takeaways
- Place plugins in a top-level `Plugins/` directory alongside `Sources/` and `Tests/`; each plugin is its own folder with a `plugin.swift` entry point.
- Use in-build commands when outputs are predictable (code generation from inputs); use pre-build commands when they are not (string extraction) — but beware of build-time cost.
- Custom commands run in a sandbox; use `.writeToPackageDirectory(reason:)` permission for plugins that need to write output to the project root.
- Plugin packages with `.plugin(name:targets:)` products can be shared as regular SwiftPM dependencies, making workflow automation reusable across projects.

---
_Source: WWDC22 Session 110401 page (abstract, chapter summaries, code samples, and resource links)._
