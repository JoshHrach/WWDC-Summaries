# Debug GPU-side Errors in Metal
**WWDC20 · Session 10616** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10616/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session introduces two new Metal debugging tools in Xcode 12 designed to close the gap between the minimal information provided by GPU-side errors and the rich diagnostics available for CPU-side API errors: Enhanced Command Buffer Errors and the Shader Validation layer.

GPU-side errors — out-of-bounds memory accesses, null texture access, invalid resource residency, and infinite loop timeouts — historically produced only a generic failure message with no encoder, draw call, or shader-level context. Enhanced Command Buffer Errors narrows the failure to the encoder level with low overhead, making it viable even in shipped apps. Shader Validation instruments Metal shaders themselves to detect undefined behavior at the draw call and even source-line level, surfacing bugs that never cause a command buffer fault (e.g., out-of-bounds reads that land in adjacent allocations).

The session includes a live demo using Metal Performance Shaders ray-tracing sample code, where Shader Validation finds a one-character typo in an array index calculation that caused visual corruption without triggering any command buffer error.

## Key Topics

**GPU-Side Error Categories**
Common GPU errors: out-of-bounds device/constant memory access, out-of-bounds threadgroup memory access, null texture resource access, invalid resource residency (missing `useResource` with argument buffers), and infinite loop / timeout. These can cause command buffer faults, visual corruption, or silent data corruption.

**Enhanced Command Buffer Errors (New)**
Improves the command buffer error mechanism by reporting encoder-level status at the time of a fault. Each encoder reports one of five states: `completed`, `pending`, `faulted` (directly caused the fault), `affected` (possibly caused by a parallel fault), or `unknown`. Enabled per-command-buffer via `MTLCommandBufferDescriptor.errorOptions = .encoderExecutionStatus`. Low overhead — suitable to leave enabled even in shipped apps (verify performance impact per device/workload). Works with existing Metal Debugger labels and debug signposts for cross-tool correlation.

**Shader Validation Layer (New)**
A GPU-side instrumentation layer similar to API Validation, but running on the GPU. Instruments Metal shaders to detect out-of-bounds device/constant memory, out-of-bounds threadgroup memory, and null texture access. Prevents the undefined operation from occurring and generates a log with encoder label, function name, file URL, and line number (if debug symbols are present). Enabled in Xcode via Scheme → Diagnostics → Metal → Shader Validation. Also supports a Metal Diagnostics breakpoint to pause at the first error. High performance and memory impact — development/QA only; do not ship with users. Process-wide: affects all Metal commands including UI rendering.

**Automated/Non-Xcode Use**
Both tools can be enabled via environment variables (`MTL_DEBUG_LAYER`, `MTL_SHADER_VALIDATION`) for automated testing pipelines. The `MTLCommandBuffer.logs` property (new) provides programmatic access to Shader Validation errors in a completion handler. System log access: `log stream --predicate "subsystem = 'com.apple.Metal' and category = 'GPUDebug'"`.

**Debugging Workflow**
Detect → Locate → Classify → Fix. Enhanced Command Buffer Errors handles Detect + Locate (encoder level). Shader Validation adds Classify (draw call / shader line level). Combined with API Validation, this covers the full range of Metal errors.

**Best Practices**
- Use asynchronous pipeline compilation to offset Shader Validation's increased compile time.
- Enable debug symbols (`-g` flag or Xcode debug scheme) for file/line information in shader backtraces.
- Use the `#line` preprocessor directive when compiling Metal libraries from source at runtime.
- Do not disable individual checks unless you've already handled that class of error; selective disabling reduces coverage.
- Always use `maxTotalThreadsPerThreadgroup` and `threadExecutionWidth` from compute pipeline state — these can differ when Shader Validation is active.

## APIs & Frameworks

### Metal — Enhanced Command Buffer Errors
- `MTLCommandBufferDescriptor` — new descriptor-based API for creating command buffers **[NEW]**
- `MTLCommandBufferDescriptor.errorOptions` — set to `.encoderExecutionStatus` to enable enhanced errors **[NEW]**
- `MTLCommandQueue.makeCommandBuffer(descriptor:)` — creates command buffer with custom options **[NEW]**
- `MTLCommandBufferEncoderInfoErrorKey` — key to access `[MTLCommandBufferEncoderInfo]` from `error.userInfo` **[NEW]**
- `MTLCommandBufferEncoderInfo` — per-encoder error info object **[NEW]**
- `MTLCommandBufferEncoderInfo.label` — encoder label string
- `MTLCommandBufferEncoderInfo.debugSignposts` — array of debug signpost strings
- `MTLCommandBufferEncoderInfo.errorState` — `.completed`, `.pending`, `.faulted`, `.affected`, `.unknown` **[NEW]**

### Metal — Shader Validation
- `MTLCommandBuffer.logs` — array of `MTLFunctionLog` objects for Shader Validation errors **[NEW]**
- `MTLFunctionLog` — contains encoder label, debug location, and description **[NEW]**
- `MTLFunctionLog.encoderLabel` — label of the faulting encoder
- `MTLFunctionLog.debugLocation` — `MTLFunctionLogDebugLocation` with file URL, function name, line, column **[NEW]**
- `MTLFunctionLog.debugLocation.functionName` — name of the faulting Metal function
- `MTLFunctionLog.debugLocation.line` / `.column` — source location of the error
- `MTLCommandBuffer.addCompletedHandler(_:)` — handler where `.logs` is valid after completion

### Environment Variables (New in macOS/iOS 14)
- `MTL_DEBUG_LAYER` — enables API Validation (non-zero value)
- `MTL_SHADER_VALIDATION` — enables Shader Validation (non-zero value) **[NEW]**
- `MTL_SHADER_VALIDATION_TEXTURE_USAGE` — set to `0` to disable null texture checks **[NEW]**

### Xcode Integration
- Scheme → Diagnostics → Metal → Shader Validation checkbox **[NEW]**
- Metal Diagnostics breakpoint (type: System Frameworks, category: Metal Diagnostics) **[NEW]**
- GPU backtrace view — shows recorded GPU call stack at time of error
- Shader annotation — inline source annotation showing error type and location

## Code Highlights

Enabling Enhanced Command Buffer Errors:
```swift
let desc = MTLCommandBufferDescriptor()
desc.errorOptions = .encoderExecutionStatus
let commandBuffer = commandQueue.makeCommandBuffer(descriptor: desc)
```

Processing encoder-level error information:
```swift
if let error = commandBuffer.error as NSError?,
   let encoderInfos = error.userInfo[MTLCommandBufferEncoderInfoErrorKey]
       as? [MTLCommandBufferEncoderInfo] {
    for info in encoderInfos {
        print(info.label + info.debugSignposts.joined())
        if info.errorState == .faulted {
            print(info.label + " faulted!")
        }
    }
}
```

Reading Shader Validation logs programmatically:
```swift
commandBuffer.addCompletedHandler { commandBuffer in
    for log in commandBuffer.logs {
        let encoderLabel = log.encoderLabel ?? "Unknown Label"
        print("Faulting encoder \"\(encoderLabel)\"")
        guard let debugLocation = log.debugLocation,
              let functionName = debugLocation.functionName else { return }
        print("Faulting function \(functionName):\(debugLocation.line):\(debugLocation.column)")
    }
}
```

## Takeaways
- Enhanced Command Buffer Errors is low overhead and appropriate to leave enabled even in shipped apps to improve telemetry; it narrows GPU faults to the encoder level using existing labels and signposts.
- Shader Validation detects undefined behavior including cases that never cause a command buffer fault (out-of-bounds reads that land in adjacent allocations) — always test with it during development and QA.
- The two tools are complementary: Enhanced Command Buffer Errors works for all GPU error types; Shader Validation adds draw-call and source-line precision for the subset of errors it instruments.
- Both tools support programmatic (non-Xcode) usage via environment variables and the new `MTLCommandBuffer.logs` API, enabling integration into automated CI/QA pipelines.

---
_Source: WWDC20 Session 10616 page (abstract, chapter summaries, code samples, and resource links)._
