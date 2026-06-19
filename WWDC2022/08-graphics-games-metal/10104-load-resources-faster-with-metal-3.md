# Load Resources Faster with Metal 3
**WWDC22 · Session 10104** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10104/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
Metal 3 introduces Fast Resource Loading, a new asynchronous API for loading assets (textures, geometry, audio) directly from SSD storage with high throughput and low latency. Unlike traditional blocking file I/O, Fast Resource Loading uses dedicated IO command queues and command buffers that execute concurrently with GPU render and compute work, fully exploiting Apple silicon's unified memory architecture and fast NVMe storage.

The API is designed around a "set-it-and-forget-it" workflow: the CPU thread issues load commands and moves on without waiting; synchronization with GPU work happens through Metal shared events. Prioritization, cancellation, and asset compression are all first-class features of the system.

## Key Topics
- **Fast Resource Loading overview** — asynchronous, concurrent asset loading that decouples I/O from CPU and GPU execution; designed for sparse texture tile streaming, geometry, and audio
- **Three-step workflow** — (1) create an `MTLIOFileHandle` for the file, (2) create an `MTLIOCommandQueue` and `MTLIOCommandBuffer` and encode load commands, (3) synchronize with GPU work via `MTLSharedEvent`
- **IO command types** — `loadTexture` (sparse texture tiles), `loadBuffer` (geometry/scene data), `loadBytes` (CPU-accessible memory for audio, etc.)
- **Concurrency model** — commands within an IO command buffer execute concurrently; multiple IO command buffers execute concurrently and can complete out of order; maximizes storage hardware throughput
- **Cancellation** — `tryCancel()` on an `MTLIOCommandBuffer` to cancel speculative preloads at command-buffer granularity; check `status` property afterward
- **Priority queues** — `MTLIOPriority.high / .normal / .low` on `MTLIOCommandQueueDescriptor`; high-priority queues get lower latency (useful for audio); priority is immutable after queue creation
- **Asset compression** — offline compression with `MTLIOCreateCompressionContext` / `MTLIOCompressionContextAppendData` / `MTLIOFlushAndDestroyCompressionContext`; inline decompression at load time; supported codecs: LZ4, ZLib, LZBitmap, LZFSE, LZMA; custom compression also possible
- **Sparse texture page sizes** — Metal 3 adds 64K and 256K sparse tile sizes in addition to the existing 16K, enabling coarser-grained streaming that better saturates storage hardware
- **Tooling** — Metal System Trace in Instruments (Xcode 14) shows IO command buffer encoding and execution timing correlated with CPU/GPU; Metal Debugger shows all IO API calls and a new Dependency Viewer visualizes IO-to-GPU synchronization edges

## APIs & Frameworks
**Metal (Metal 3 — Fast Resource Loading)** **[NEW]**
- `MTLDevice.makeIOHandle(url:)` **[NEW]** — creates `MTLIOFileHandle` for an uncompressed file
- `MTLDevice.makeIOHandle(url:compressionMethod:)` **[NEW]** — creates `MTLIOFileHandle` for a compressed file
- `MTLIOFileHandle` **[NEW]** — opaque handle to an open file, used as the source in load commands
- `MTLIOCommandQueueDescriptor` **[NEW]** — descriptor for IO command queue; properties: `type` (`.concurrent` or `.serial`), `priority` (`MTLIOPriority`)
- `MTLIOCommandQueueType` **[NEW]** — `.concurrent` (default), `.serial`
- `MTLIOPriority` **[NEW]** — `.high`, `.normal`, `.low`
- `MTLDevice.makeIOCommandQueue(descriptor:)` **[NEW]** — creates `MTLIOCommandQueue`
- `MTLIOCommandQueue` **[NEW]** — queue for IO command buffers
- `MTLIOCommandQueue.makeCommandBuffer()` **[NEW]** — creates `MTLIOCommandBuffer`
- `MTLIOCommandBuffer` **[NEW]** — encodes and submits IO load commands; properties: `status: MTLIOStatus`
- `MTLIOCommandBuffer.load(_:slice:level:size:sourceBytesPerRow:sourceBytesPerImage:destinationOrigin:sourceHandle:sourceHandleOffset:)` **[NEW]** — `loadTexture` command
- `MTLIOCommandBuffer.load(_:offset:size:sourceHandle:sourceHandleOffset:)` **[NEW]** — `loadBuffer` command
- `MTLIOCommandBuffer.loadBytes(_:size:sourceHandle:sourceHandleOffset:)` **[NEW]** — `loadBytes` command
- `MTLIOCommandBuffer.waitForEvent(_:value:)` **[NEW]** — wait for a `MTLSharedEvent` signal before executing subsequent commands
- `MTLIOCommandBuffer.signalEvent(_:value:)` **[NEW]** — signal a `MTLSharedEvent` after all prior commands complete
- `MTLIOCommandBuffer.commit()` **[NEW]** — submit the command buffer to the queue for execution
- `MTLIOCommandBuffer.tryCancel()` **[NEW]** — attempt to cancel execution; cancellation is best-effort
- `MTLIOStatus` **[NEW]** — `.pending`, `.running`, `.complete`, `.cancelled`, `.error`

**Metal (Compression)** **[NEW]**
- `MTLIOCompressionMethod` **[NEW]** — enum: `.zlib`, `.lzfse`, `.lz4`, `.lzBitmap`, `.lzma`
- `MTLIOCreateCompressionContext(_:_:_:)` **[NEW]** — creates an offline compression context; parameters: output file path, compression method, chunk size
- `MTLIOCompressionContextAppendData(_:_:_:)` **[NEW]** — appends data to the compression context
- `MTLIOFlushAndDestroyCompressionContext(_:)` **[NEW]** — finalizes and writes the compressed file

**Metal (existing, used for synchronization)**
- `MTLSharedEvent` — existing; used to synchronize IO command buffers with GPU render/compute command buffers
- `MTLDevice.makeSharedEvent()` — creates a shared event

**Sparse Textures (Metal 3 additions)**
- New sparse tile sizes: 64K and 256K in addition to existing 16K

## Code Highlights
Create file handle and load a texture:
```swift
let fileHandle = try device.makeIOHandle(url: filePath)
let ioCommandBuffer = ioCommandQueue.makeCommandBuffer()
ioCommandBuffer.load(texture, slice: 0, level: 0, size: size,
                     sourceBytesPerRow: bytesPerRow, sourceBytesPerImage: bytesPerImage,
                     destinationOrigin: destOrigin,
                     sourceHandle: fileHandle, sourceHandleOffset: 0)
ioCommandBuffer.commit()
```

Synchronize IO with GPU rendering using a shared event:
```swift
let sharedEvent = device.makeSharedEvent()
ioCommandBuffer.waitForEvent(sharedEvent, value: waitVal)
// encode load commands
ioCommandBuffer.signalEvent(sharedEvent, value: signalVal)
ioCommandBuffer.commit()
// GPU command buffer waits for sharedEvent before rendering
```

Create a high-priority queue for audio:
```swift
let descriptor = MTLIOCommandQueueDescriptor()
descriptor.priority = .high
let audioQueue = try device.makeIOCommandQueue(descriptor: descriptor)
```

Offline asset compression:
```swift
let ctx = MTLIOCreateCompressionContext(outputPath, .zlib, 64 * 1024)
MTLIOCompressionContextAppendData(ctx, fileData.bytes, fileData.length)
MTLIOFlushAndDestroyCompressionContext(ctx)
// Later, open compressed file:
let handle = try device.makeIOHandle(url: compressedPath, compressionMethod: .zlib)
```

## Takeaways
- Fast Resource Loading enables fully asynchronous, multi-threaded asset loading that maximizes SSD throughput on Apple silicon without blocking the CPU thread.
- Synchronization with the GPU is explicit via `MTLSharedEvent` — the CPU thread never needs to block waiting for I/O to complete.
- Use separate high-priority IO queues for latency-sensitive data like audio; use normal-priority queues for background texture/geometry streaming.
- Asset compression reduces disk footprint and can be applied offline; Metal handles inline decompression transparently at load time.

---
_Source: WWDC22 Session 10104 page (abstract, chapter summaries, code samples, and resource links)._
