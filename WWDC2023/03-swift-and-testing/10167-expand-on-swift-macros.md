# Expand on Swift Macros
**WWDC23 · Session 10167** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10167/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10, tvOS 17, visionOS 1

## Overview
This session provides a deep technical look at Swift macros — a new language extension mechanism introduced in Swift 5.9 that lets developers add custom code transformations as distributable Swift packages, without modifying the compiler. Macros reduce boilerplate by transforming a small use site into a more complex piece of code at compile time, using a compiler plug-in that runs in a secure sandbox.

The session covers macro design philosophy (visible, type-safe, additive, inspectable), the translation model (plug-in process, SwiftSyntax trees), all seven macro roles, the implementation process using `SwiftSyntax`/`SwiftSyntaxBuilder`, error diagnostics with `DiagnosticMessage`, name collision hygiene, and unit testing with `assertMacroExpansion`.

The companion session "Write Swift Macros" (10166) covers practical Xcode tooling and workflow.

## Key Topics

- **Design philosophy** — Macros use `#` (freestanding) or `@` (attached) sigils for visibility; inputs and outputs are fully type-checked; expansions are additive only (can't remove or change existing code); expansions are inspectable in Xcode via right-click expand, debugger step-in, and breakpoints.
- **Translation model** — Compiler extracts a macro use, sends it to a compiler plug-in process running in a sandbox; plug-in returns a SwiftSyntax expansion; compiler inserts it back and compiles everything together.
- **Macro roles** — Two freestanding: `@freestanding(expression)` (produces a value), `@freestanding(declaration)` (produces declarations). Five attached: `@attached(peer)` (adds declarations alongside), `@attached(accessor)` (installs get/set/willSet/didSet), `@attached(memberAttribute)` (adds attributes to members), `@attached(member)` (adds new members including stored properties/cases), `@attached(conformance)` (adds protocol conformances). Roles can be composed on a single macro.
- **Implementation** — Macro declaration in library with `= #externalMacro(module:type:)`; implementation type in separate compiler plug-in module; conforms to role protocols (`MemberMacro`, `AccessorMacro`, `PeerMacro`, etc.); `SwiftSyntax` represents code as typed syntax node trees; `SwiftSyntaxBuilder` provides convenience for generating code.
- **Diagnostics** — `MacroExpansionContext.diagnose(_:)` with `Diagnostic` instances; custom `DiagnosticMessage` protocol types; Fix-It support; highlights; attached notes.
- **Name hygiene** — Macros are not hygienic by design (intentional for member access); use `MacroExpansionContext.makeUniqueName()` for generated local names; macros must declare introduced names in role attributes via `names:` parameter (specifiers: `overloaded`, `prefixed(_:)`, `suffixed(_:)`, `named(_:)`, `arbitrary`).
- **Testing** — Macro plug-in is a normal Swift module; use `assertMacroExpansion` from `SwiftSyntaxMacrosTestSupport`.

## APIs & Frameworks

**Swift Macros (language feature) [NEW]**
- `@freestanding(expression)` role attribute **[NEW]**
- `@freestanding(declaration)` role attribute **[NEW]**
- `@attached(peer)` role attribute **[NEW]**
- `@attached(accessor)` role attribute **[NEW]**
- `@attached(memberAttribute)` role attribute **[NEW]**
- `@attached(member, names:)` role attribute **[NEW]**
- `@attached(conformance)` role attribute **[NEW]**
- `macro` keyword **[NEW]** — declares a macro
- `#externalMacro(module:type:)` **[NEW]** — links declaration to plug-in implementation

**SwiftSyntax** (Swift package, updated for macros)
- `SwiftSyntax` module — typed syntax node tree types (e.g., `StructDeclSyntax`, `AttributeListSyntax`, `MemberDeclBlockSyntax`, `DeclSyntax`, `ExprSyntax`, `StmtSyntax`, `TokenSyntax`)
- `SwiftSyntaxMacros` module — macro role protocols: `ExpressionMacro`, `DeclarationMacro`, `PeerMacro`, `AccessorMacro`, `MemberAttributeMacro`, `MemberMacro`, `ConformanceMacro`
- `SwiftSyntaxBuilder` module — convenience APIs for constructing syntax trees; string literal `DeclSyntax`/`ExprSyntax`/`StmtSyntax` init with `\(literal:)` interpolation
- `MacroExpansionContext` protocol — `makeUniqueName(_:)`, `diagnose(_:)`, `location(of:)` (returns `AbstractSourceLocation`)
- `AbstractSourceLocation` — provides file/line/column `ExprSyntax` nodes for diagnostics
- `Diagnostic` struct — `init(node:message:)`, supports Fix-Its, highlights, notes
- `DiagnosticMessage` protocol — `message: String`, `severity: DiagnosticSeverity`, `diagnosticID: MessageID`
- `DiagnosticSeverity` enum — `.error`, `.warning`
- `MessageID` struct — `init(domain:id:)`
- `FixIt` — attached to `Diagnostic` for Xcode Fix button

**SwiftSyntaxMacrosTestSupport**
- `assertMacroExpansion(_:expandingMacrosWith:expectedExpandedSource:)` **[NEW]** — unit test assertion for macro output

## Code Highlights

Declaring a freestanding expression macro:
```swift
@freestanding(expression)
macro unwrap<Wrapped>(_ expr: Wrapped?, message: String) -> Wrapped
    = #externalMacro(module: "MyLibMacros", type: "UnwrapMacro")
```

Composing multiple roles on one macro:
```swift
@attached(accessor)
@attached(memberAttribute)
@attached(member, names: named(dictionary), named(init))
@attached(conformance)
macro DictionaryStorage(key: String? = nil) = #externalMacro(...)
```

Emitting a custom diagnostic from a macro implementation:
```swift
func expansion(of attribute: AttributeSyntax,
               providingMembersOf declaration: some DeclGroupSyntax,
               in context: some MacroExpansionContext) throws -> [DeclSyntax] {
    guard declaration.is(StructDeclSyntax.self) else {
        context.diagnose(Diagnostic(
            node: attribute,
            message: MyDiagnostic.notAStruct))
        return []
    }
    // ...
}
```

Using `makeUniqueName` to avoid name collisions:
```swift
let uniqueName = context.makeUniqueName("wrappedValue")
```

## Takeaways

- Macros are declared in a library with `macro` + role attributes and implemented in a separate compiler plug-in module using SwiftSyntax — distribute both in one Swift package.
- Compose multiple roles on one macro (e.g., `@attached(accessor)` + `@attached(member)` + `@attached(conformance)`) to progressively eliminate boilerplate with a single attribute.
- Emit rich `Diagnostic` errors from macro implementations to guide developers — include Fix-Its and source highlights for a first-class experience.
- Write unit tests with `assertMacroExpansion` from the start; test-driven development is especially effective for macros.

---
_Source: WWDC23 Session 10167 page (abstract, chapter summaries, code samples, and resource links)._
