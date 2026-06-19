# Write Swift macros
**WWDC23 · Session 10166** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10166/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1, Linux, Windows (Swift 5.9)

## Overview
This hands-on code-along session teaches Swift macro authoring from first principles: declaring macros with the `@freestanding` and `@attached` roles, implementing them as SwiftSyntax compiler plug-ins in a separate Swift package, testing them with `assertMacroExpansion`, debugging syntax trees in the Xcode debugger, emitting typed compiler diagnostics, and publishing macros as Swift packages. The running example builds a `@SlopeSubset` attached member macro that auto-generates a failable initializer and computed property for enum subsets — eliminating hand-written boilerplate while preserving compile-time type safety.

## Key Topics

**How Macros Work**
- Macros are compile-time Swift programs implemented as SwiftSyntax compiler plug-ins in a separate Swift package target
- Macro declaration looks like a function: defines name, generic parameters, parameter types, return type, and roles
- Compiler type-checks arguments before expansion — unlike C preprocessor macros; invalid arguments produce type errors, not bad expansions
- Expansion process: compiler serializes macro expression to plug-in → plug-in parses to `SwiftSyntaxTree` → transforms → serializes back → compiler replaces call site

**Macro Roles**
- `@freestanding(expression)` — used with `#`; replaces an expression at the call site
- `@freestanding(declaration)` — used with `#`; expands to one or more declarations
- `@attached(member, names:)` — used with `@`; adds new members to the attached type
- `@attached(accessor)`, `@attached(memberAttribute)`, `@attached(peer)` — covered in depth in "Expand on Swift macros" (10167)

**Freestanding Expression Macro (`#stringify`)**
- Template included in Xcode's "Swift Macro" package template
- Declaration: `@freestanding(expression) public macro stringify<T>(_ value: T) -> (T, String) = #externalMacro(module:type:)`
- `#externalMacro(module:type:)` points the compiler to the implementing type in the compiler plug-in module
- Implementation: conform to `ExpressionMacro`; implement `static func expansion(of:in:) -> ExprSyntax`; return `ExprSyntax` (auto-parsed from string literals with `\()` interpolation)

**Attached Member Macro (`@SlopeSubset` / `@EnumSubset`)**
- Declaration: `@attached(member, names: named(init)) public macro SlopeSubset() = #externalMacro(...)`
- `names:` parameter declares what member names the macro will introduce (required for `member` role)
- Implementation: conform to `MemberMacro`; implement `static func expansion(of:providingMembersOf:in:) throws -> [DeclSyntax]`
- Return `[DeclSyntax]` — array of new member declarations to add to the type
- Use `InitializerDeclSyntax`, `SwitchExprSyntax`, `SwitchCaseSyntax` result-builder APIs to build AST nodes

**Inspecting the Syntax Tree**
- Set a breakpoint inside `expansion`; run the test target; `po enumDecl` in LLDB to print the SwiftSyntax tree
- Tree navigation: `enumDecl.memberBlock.members` → `compactMap { $0.decl.as(EnumCaseDeclSyntax.self) }` → `flatMap { $0.elements }`

**Error Emission**
- Throw any `Error` from the expansion function → error appears at the `@` attribute call site
- For custom placement, warnings, or Fix-Its: use `context.addDiagnostic(Diagnostic(...))`
- `DiagnosticSpec` in tests (`assertMacroExpansion(diagnostics:)`) verifies emitted error messages
- Pattern: define an `enum MyMacroError: CustomStringConvertible, Error` with localized descriptions

**Testing**
- Use `assertMacroExpansion(_:expandedSource:macros:)` from `SwiftSyntaxMacrosTestSupport`
- `macros:` dictionary maps macro name strings to `Macro.Type` implementations
- Tests are deterministic (no side effects) and fast — ideal for TDD while building macros
- Run tests via standard Xcode Test navigator

**Generic Macros**
- Add generic type parameters to the macro declaration: `@attached(member, ...) public macro EnumSubset<Superset>() = ...`
- At runtime, retrieve the generic argument from the attribute syntax: `attribute.attributeName.as(SimpleTypeIdentifierSyntax.self)?.genericArgumentClause?.arguments.first?.argumentType`

## APIs & Frameworks

**Swift Macros (language)**
- `macro` keyword **[NEW]** — declares a Swift macro
- `@freestanding(expression)` macro role **[NEW]**
- `@freestanding(declaration)` macro role **[NEW]**
- `@attached(member, names:)` macro role **[NEW]**
- `#externalMacro(module:type:)` **[NEW]** — links declaration to plug-in implementation
- `CompilerPlugin` protocol **[NEW]** — `@main` entry point for macro plug-in; `providingMacros: [Macro.Type]`

**SwiftSyntax / SwiftSyntaxMacros**
- `ExpressionMacro` protocol **[NEW]** — implement for `@freestanding(expression)` macros
- `ExpressionMacro.expansion(of:in:) -> ExprSyntax` **[NEW]**
- `MemberMacro` protocol **[NEW]** — implement for `@attached(member)` macros
- `MemberMacro.expansion(of:providingMembersOf:in:) throws -> [DeclSyntax]` **[NEW]**
- `FreestandingMacroExpansionSyntax` — node type for `#macro(...)` call sites; `.argumentList`
- `MacroExpansionContext` — compiler communication; `addDiagnostic(_:)`
- `DeclSyntax` — type-erased declaration syntax node
- `ExprSyntax` — type-erased expression syntax node
- `EnumDeclSyntax` — enum declaration node; `.memberBlock.members`
- `EnumCaseDeclSyntax` — enum case declaration node; `.elements`
- `EnumCaseElementSyntax` — individual enum element; `.identifier`
- `InitializerDeclSyntax(_:)` result-builder init **[NEW]**
- `SwitchExprSyntax(_:)` result-builder init **[NEW]**
- `SwitchCaseSyntax` **[NEW]**
- `SimpleTypeIdentifierSyntax` — type name with optional generic arguments
- `AttributeSyntax` — represents an `@MacroName(...)` attribute

**SwiftSyntaxMacrosTestSupport**
- `assertMacroExpansion(_:expandedSource:diagnostics:macros:)` **[NEW]** — unit test assertion for macro expansion
- `DiagnosticSpec(message:line:column:)` **[NEW]** — expected diagnostic specification

## Code Highlights

Declaring a freestanding expression macro:
```swift
@freestanding(expression)
public macro stringify<T>(_ value: T) -> (T, String) =
    #externalMacro(module: "WWDCMacros", type: "StringifyMacro")
```

Implementing it:
```swift
public struct StringifyMacro: ExpressionMacro {
    public static func expansion(
        of node: some FreestandingMacroExpansionSyntax,
        in context: some MacroExpansionContext
    ) -> ExprSyntax {
        guard let argument = node.argumentList.first?.expression else {
            fatalError("compiler bug: no arguments")
        }
        return "(\(argument), \(literal: argument.description))"
    }
}
```

Declaring an attached member macro:
```swift
@attached(member, names: named(init))
public macro SlopeSubset() =
    #externalMacro(module: "WWDCMacros", type: "SlopeSubsetMacro")
```

Implementing it (member generation):
```swift
public struct SlopeSubsetMacro: MemberMacro {
    public static func expansion(
        of attribute: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        guard let enumDecl = declaration.as(EnumDeclSyntax.self) else {
            throw SlopeSubsetError.onlyApplicableToEnum
        }
        let elements = enumDecl.memberBlock.members
            .compactMap { $0.decl.as(EnumCaseDeclSyntax.self) }
            .flatMap { $0.elements }
        let initializer = try InitializerDeclSyntax("init?(_ slope: Slope)") {
            try SwitchExprSyntax("switch slope") {
                for element in elements {
                    SwitchCaseSyntax("case .\(element.identifier): self = .\(element.identifier)")
                }
                SwitchCaseSyntax("default: return nil")
            }
        }
        return [DeclSyntax(initializer)]
    }
}
```

Testing with `assertMacroExpansion`:
```swift
assertMacroExpansion(
    """
    @SlopeSubset
    enum EasySlope { case beginnersParadise; case practiceRun }
    """,
    expandedSource: """
    enum EasySlope {
        case beginnersParadise; case practiceRun
        init?(_ slope: Slope) {
            switch slope {
            case .beginnersParadise: self = .beginnersParadise
            case .practiceRun: self = .practiceRun
            default: return nil
            }
        }
    }
    """,
    macros: ["SlopeSubset": SlopeSubsetMacro.self]
)
```

## Takeaways
- Always write unit tests for macros using `assertMacroExpansion` before or alongside the implementation — expanded code is deterministic and text-comparable, making TDD natural.
- Print the SwiftSyntax tree in the debugger (`po node`) to map from "what I see in source code" to "which SwiftSyntax properties I need to access" — far faster than reading documentation cold.
- Throw a typed `Error` from `expansion` to surface clear error messages at the macro call site; use `context.addDiagnostic` when you need specific source locations, warnings, or Fix-Its.
- Generic macro parameters (`macro EnumSubset<Superset>()`) let you build reusable libraries of macros that are not hardcoded to specific types.

---
_Source: WWDC23 Session 10166 page (abstract, chapters, transcript, and code samples)._
