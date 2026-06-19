# Explore Swift Performance
**WWDC24 · Session 10217** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10217/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, tvOS 18, watchOS 11, visionOS 2

## Overview
Presented by John McCall, this session builds a systematic mental model for Swift performance centered on four low-level principles: function calls, memory allocation, memory layout, and value copying. Unlike a tooling session, this is a conceptual foundation talk — understanding which Swift constructs imply which costs so you can make informed design decisions and recognize when the optimizer is helping you.

The session explains not only the costs themselves but also optimization potential: how the Swift optimizer eliminates costs under favorable conditions, how you can write code that maximizes optimizer effectiveness, and how to build automated measurements to verify the optimizer keeps working as expected.

## Key Topics

### Function Calls
Every call has four associated costs: argument setup, function address resolution (static vs. dynamic dispatch), local-state allocation (the call frame on the C stack), and optimization inhibition. Static dispatch lets the compiler inline and specialize; dynamic dispatch is required for polymorphism. Protocol requirements declared in the main protocol body use dynamic dispatch; methods declared in protocol extensions use static dispatch — a critical and non-obvious distinction.

### Memory Allocation
Three kinds of memory: global (cheapest, fixed lifetime), stack (very cheap, scoped), and heap (flexible but substantially more expensive). Class instances use the heap. Some features — escaping closures, `any` protocol values with large payloads — also trigger heap allocation. CallFrame allocation (subtracting from the stack pointer) is essentially free because the subtraction instruction already runs for every function.

### Memory Layout: Inline vs. Out-of-Line Storage
Structs, tuples, and enums store all their contents **inline** in their container. Classes use **out-of-line** storage: the container holds a pointer to the heap object. `MemoryLayout<T>.size` measures only the inline representation. Copying a struct copies all stored properties recursively (including retaining any reference-typed fields); copying a class just copies (and retains) the pointer. Large structs can therefore be more expensive to copy than class instances, making the struct-vs-class trade-off situation-dependent.

### Value Copying and Ownership
Three ownership interactions: **consuming** (transfers ownership, may avoid a copy), **mutating** (temporarily takes ownership and returns a new value), and **borrowing** (read-only, no copy needed). The compiler inserts defensive copies when it cannot prove simultaneous mutation is impossible — common with class properties. Use `consume` to explicitly transfer ownership and eliminate copies. Swift is actively improving this area with optimizer improvements and explicit borrow annotations.

### Dynamically-Sized Types
Types like `Foundation.URL` and generic type parameters have layouts unknown at compile time. In most containers this is handled dynamically at runtime with no heap cost. In containers that must have a constant size (global variables, local variables on the C CallFrame), Swift allocates extra heap storage lazily. Generic type parameters constrained to `AnyObject` always have pointer representation, enabling much more efficient code.

### Async Functions
Async functions keep local state on a separate async stack (slabs of memory held by the task), not the C stack. Local state that does not cross an `await` point can still go in the C CallFrame. The function is split into partial functions at each potential suspension point; suspension simply returns to the concurrency runtime without blocking the thread. The task-local allocator is significantly faster than `malloc` because it serves a single task and uses stack discipline.

### Closures
Function values are a (function pointer, context pointer) pair. Non-escaping closures can stack-allocate their context — cheap. Escaping closures require heap-allocated, reference-counted context objects. Local `var`s captured by escaping closures must also be heap-allocated (wrapped in a reference box) so their lifetime can extend beyond the enclosing scope.

### Generics and Protocol Types
Generic functions with protocol constraints pass a protocol witness table as a hidden extra argument. When the concrete type is known at the call site, the optimizer can specialize the function (generating a concrete copy with static dispatch). `any Protocol` existentials carry their own type metadata and witness table inline, and use a 3-pointer storage buffer — values that fit are stored inline; larger values are heap-allocated. Heterogeneous `[any Protocol]` arrays prevent specialization and force per-element dynamic dispatch.

## APIs & Frameworks

**Swift Language / Compiler Concepts**
- Static dispatch — compile-time call resolution; enables inlining and specialization
- Dynamic dispatch — virtual/witness table lookup; enables polymorphism
- Protocol requirements (main body) — use dynamic dispatch
- Protocol extension methods — use static dispatch
- `consume` operator — explicitly transfer ownership to avoid copies
- `consuming` parameter ownership modifier — callee takes ownership
- `inout` parameter ownership modifier — mutating, ownership returned after call
- `borrowing` parameter ownership modifier — read-only, no copy
- `MemoryLayout<T>.size` / `MemoryLayout.size(ofValue:)` — measures inline representation size
- Existential container (`any Protocol`) — 3-word `OpaqueValueStorage` + `TypeMetadata*` + `WitnessTable*`
- Protocol Witness Table — per-conformance function pointer table
- Generic specialization — optimizer generates concrete copies of generic functions
- Async partial functions — async functions split at each `await` into partial C functions
- Async task slab allocator — task-local stack allocator for async local state
- Non-escaping closure context — stack-allocated
- Escaping closure context — heap-allocated, reference-counted
- `Box<T>` pattern — how captured `var`s become heap-allocated in escaping closures

**Related Tools**
- Instruments — recommended for top-down performance investigation
- `-O` / Whole-Module Optimization — enable for specialization and inlining

## Code Highlights

Static vs. dynamic dispatch — protocol requirement vs. extension method:
```swift
protocol DataModel {
    func update(from source: DataSource)           // dynamic dispatch
}
extension DataModel {
    func update(from source: DataSource) {         // static dispatch
        self.update(from: source, quickly: true)
    }
}
```

Copying with consume to avoid defensive copies:
```swift
func makeArray() {
    var array = [ 1.0, 2.0 ]
    var array2 = consume array   // transfers ownership, no retain
}
```

Async function — local state on async stack, split at each await:
```swift
func awaitAll(tasks: [Task<Int, Never>]) async -> [Int] {
    var results = [Int]()
    for task in tasks {
        results.append(await task.value)
    }
    return results
}
```

Generic function specialization:
```swift
func updateAll<Model: DataModel>(models: [Model], from source: DataSource) {
    for model in models { model.update(from: source) }
}
var myModels: [MyDataModel]
updateAll(models: myModels, from: source)
// Optimizer implicitly generates:
// func updateAll_specialized(models: [MyDataModel], from source: DataSource) { ... }
```

Existential vs. generic — memory layout contrast:
```swift
struct AnyDataModel {             // what `any DataModel` looks like in C
    OpaqueValueStorage value;     // 3 pointers inline; larger values heap-allocated
    TypeMetadata *valueType;
    DataModelWitnessTable *value_is_DataModel;
}
```

## Takeaways
- Protocol methods declared in the main body use dynamic dispatch; methods in extensions use static dispatch — knowing this distinction is essential for predicting overhead.
- Prefer inline struct storage for small, frequently-created values; switch to classes when sharing, identity, or large-value copying becomes expensive.
- Use `consume` / `consuming` / `borrowing` to give the optimizer explicit ownership information and eliminate defensive copies, especially in hot paths.
- Async functions are efficient (task-local slab allocator is much faster than `malloc`), but local state that crosses `await` points cannot live in the C CallFrame — structure async code to minimize cross-suspension state.

---
_Source: WWDC24 Session 10217 page (abstract, chapter summaries, code samples, and resource links)._
