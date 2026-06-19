# Explore Swift and Java interoperability

**Session ID:** 307  
**WWDC Year:** 2025  
**Folder:** `03-swift-and-testing`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/307/

---

## Overview

This session introduces the open-source swift-java project, which enables bidirectional interoperability between Swift and Java. The session explains the motivation — many enterprise and Android codebases are written in Java or Kotlin, and developers increasingly want to share Swift logic with JVM targets or call Java libraries from Swift. It walks through the two main use cases: calling Java APIs from Swift, and exposing Swift types and functions to Java. The session covers the swift-java Swift package plugin, automatic Java wrapper generation from `.jar` files, Swift wrapper generation from Java annotations, and how the interop layer handles memory and threading across the JVM/Swift boundary.

---

## Key Topics

- Motivation for Swift–Java interop: sharing code with Android, server-side Java, and enterprise JVM stacks
- The `swift-java` open-source Swift package and its two code generation modes
- Importing Java libraries into Swift: generating Swift wrappers from a `.jar` via `swift-java import`
- Exposing Swift to Java: annotating Swift declarations and generating Java wrappers
- Memory model: `JavaObject` lifetime management and `with` scoping
- Threading: JVM thread attachment, Swift concurrency integration
- Error handling: Java exceptions surfaced as Swift `Error` types
- Supported Java versions and runtime requirements

---

## APIs & Frameworks

- **swift-java** – **[NEW]** Open-source Swift package (https://github.com/swiftlang/swift-java) providing build plugins and runtime support for Swift–Java interop.
- **`JavaKit`** – **[NEW]** Swift module (part of swift-java) providing the core `JavaObject`, `JavaClass`, `JavaEnvironment`, and primitive bridging types.
- **`JavaObject`** – **[NEW]** Swift wrapper base class for all Java objects; manages JNI global references. Instances must be used within a `JavaEnvironment` scope.
- **`JavaEnvironment`** – **[NEW]** Wraps a JNI `JNIEnv*`; attach to a Java VM with `JavaVirtualMachine.shared.environment()`.
- **`JavaVirtualMachine`** – **[NEW]** Singleton that boots and manages the embedded JVM in a Swift process.
- **`swift package java-wrap`** CLI subcommand – **[NEW]** Generates Swift wrapper types from a provided `.jar` file; output is source files you add to your Swift target.
- **`@JavaClass` / `@JavaMethod` / `@JavaField`** Swift macros – **[NEW]** Applied to Swift `class` and `func` declarations to mark them for Java wrapper generation.
- **`swift package java-export`** CLI subcommand – **[NEW]** Generates `.java` wrapper files that call into Swift via JNI for declarations annotated with `@JavaClass`.
- **Java exception bridging** – Java exceptions thrown through JNI are automatically converted to Swift `Error` values conforming to `JavaException`; catch them with standard Swift `do/catch`.
- **Swift Concurrency integration** – Async Swift functions called from Java execute on the Swift cooperative thread pool; the interop layer handles JVM thread attachment/detachment automatically.
- **`JavaArray<Element>`** – **[NEW]** Generic Swift type bridging Java arrays (e.g., `JavaArray<JavaInt>`).

---

## Code Highlights

Loading a Java class and calling a method from Swift:
```swift
import JavaKit

let jvm = try JavaVirtualMachine.shared
let env = try jvm.environment()

// Generated wrapper (via swift package java-wrap)
let list = try ArrayList<JavaString>(environment: env)
try list.add("Hello from Swift")
let size = try list.size()
print("List size: \(size)")  // 1
```

Exposing a Swift function to Java:
```swift
import JavaKit

@JavaClass("com.example.SwiftHelper")
public class SwiftHelper: JavaObject {
    @JavaMethod
    public func greet(name: String) -> String {
        return "Hello, \(name), from Swift!"
    }
}
```

Calling the generated Java wrapper from Java:
```java
SwiftHelper helper = new SwiftHelper();
String result = helper.greet("World");
System.out.println(result); // Hello, World, from Swift!
```

---

## Takeaways

- swift-java enables genuine bidirectional Swift–Java interop without writing JNI boilerplate manually; code generation handles the binding layer.
- The primary use cases are calling existing Java/Android libraries from Swift and exposing Swift business logic to JVM-based server or Android targets.
- `JavaObject` lifetimes are managed through Swift's ARC interacting with JNI global references; always keep a `JavaEnvironment` in scope.
- Swift `async`/`await` functions can be called from Java through the generated wrappers; the interop layer handles thread model differences automatically.
- swift-java is an open-source project under active development; production readiness should be evaluated on a per-use-case basis.
- Java exceptions propagate as Swift `Error` values, making error handling idiomatic on the Swift side.
