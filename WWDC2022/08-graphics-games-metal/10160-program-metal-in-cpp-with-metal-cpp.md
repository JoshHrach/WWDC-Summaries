# Program Metal in C++ with metal-cpp
**WWDC22 · Session 10160** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10160/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
`metal-cpp` is a lightweight, header-only C++ wrapper for the Metal, Foundation, and CoreAnimation Objective-C frameworks, enabling C++ games and applications to access the full Metal API without writing any Objective-C. It provides 100% coverage of the Metal API through a 1-to-1 mapping of C++ calls to Objective-C APIs, using the same C-level Objective-C runtime mechanism the compiler itself uses, so overhead is minimal. All Apple developer tools (GPU Frame Capture, Xcode debugger, Instruments) work seamlessly with metal-cpp code.

The session covers three major topics: how to set up and use metal-cpp, how to correctly manage object lifecycles with Cocoa's Manual Retain-Release (MRR) conventions, and how to architect an app that cleanly integrates C++ (metal-cpp) code with Objective-C frameworks using adapter classes and ARC bridge casts.

metal-cpp is open source under the Apache 2 License and available at developer.apple.com/metal/cpp/.

## Key Topics

### metal-cpp Setup
1. Download metal-cpp and add to the Xcode project
2. Set C++ language dialect to C++17 or higher
3. Add frameworks: Foundation, QuartzCore, Metal
4. Define three macros exactly once before importing headers: `NS_PRIVATE_IMPLEMENTATION`, `CA_PRIVATE_IMPLEMENTATION`, `MTL_PRIVATE_IMPLEMENTATION`
5. Include `<Foundation/Foundation.hpp>`, `<Metal/Metal.hpp>`, `<QuartzCore/QuartzCore.hpp>`

### Cocoa MRR Object Lifecycle Rules
- metal-cpp uses Manual Retain-Release (not ARC)
- Own objects created with `alloc`, `new`, `copy`, `mutableCopy`, or `create` prefixes
- Take ownership with `retain`; release ownership with `release`
- Never call `init` twice on an object
- Never release an object you don't own (double-free risk)
- Use `autorelease` to relinquish ownership without immediate deallocation; the `AutoreleasePool` takes ownership and releases objects when it is destroyed
- Create `NS::AutoreleasePool` for main thread and for every additional thread; create smaller-scope pools to reduce working set

### NS::SharedPtr Utility (metal-cpp)
- `NS::TransferPtr(pMRR)` — wraps a raw pointer, takes ownership without incrementing retain count; calls `release` on destruction
- `NS::RetainPtr(pMRR)` — wraps a raw pointer, increments retain count (calls `retain`); calls `release` on destruction; use to keep alive objects in AutoreleasePools
- `.get()` / `operator->()` — access underlying pointer
- Copy increases retain count; move does not affect retain count
- `.detach()` — release ownership without calling release on destruction
- Not the same as `std::shared_ptr` — no C++ stdlib dependency, no extra reference count storage

### NSZombie for Use-After-Free Debugging
- Set environment variable `NSZombieEnabled=YES` or enable in Xcode scheme
- When a deallocated object receives a message, triggers a breakpoint with "message sent to deallocated instance" console output

### Adapter Pattern: Integrating C++ and Objective-C
- **C++ calling Objective-C**: Create an Objective-C adapter class with a C++ pointer to the renderer; implement Objective-C methods that delegate to C++ methods
- **Objective-C calling C++**: Create a C++ adapter class with C++ interfaces; implement them using Objective-C method calls
- Use `__bridge` casts to move pointers between ARC and MRR worlds

### ARC Bridge Casts
- `__bridge` — cast between Objective-C and metal-cpp with no ownership transfer (toll-free bridging)
- `__bridge_retained` — cast from Objective-C (ARC) to metal-cpp (MRR), transferring ownership out of ARC; caller is now responsible for `release`
- `__bridge_transfer` — cast from metal-cpp (MRR) to Objective-C (ARC), transferring ownership to ARC

## APIs & Frameworks

**metal-cpp** (header-only C++ wrapper) **[NEW public release WWDC22]**
- `MTL::Device*` — C++ wrapper for `id<MTLDevice>`
- `MTL::CommandQueue*` — `->commandBuffer()` returns autoreleased command buffer
- `MTL::CommandBuffer*` — `->renderCommandEncoder(pRpd)`, `->commit()`, `->presentDrawable()`
- `MTL::RenderCommandEncoder*` — `->setRenderPipelineState()`, `->drawPrimitives()`, `->endEncoding()`
- `MTL::RenderPassDescriptor*`
- `MTL::RenderPipelineState*`
- `MTL::Texture*`
- `MTL::Buffer*`
- `MTL::PrimitiveTypeTriangle` — etc. (all MTL enums mirrored)
- `CA::MetalDrawable*`
- `MTK::View*`

**NS:: (Foundation wrapper)**
- `NS::AutoreleasePool::alloc()->init()` — create autorelease pool
- `NS::AutoreleasePool::release()` — drain and release pool
- `NS::TransferPtr(raw)` — **[NEW]** take ownership without retain
- `NS::RetainPtr(raw)` — **[NEW]** retain and manage lifetime
- `NS::SharedPtr<T>` — **[NEW]** smart pointer with Cocoa MRR semantics
  - `.get()`, `operator->()`, `.detach()`

**Integration Macros**
- `NS_PRIVATE_IMPLEMENTATION`
- `CA_PRIVATE_IMPLEMENTATION`
- `MTL_PRIVATE_IMPLEMENTATION`

**Objective-C Bridge Casts**
- `(__bridge T*)` — no ownership transfer
- `(__bridge_retained T*)` — ARC → MRR, transfer ownership out of ARC
- `(__bridge_transfer T*)` — MRR → ARC, transfer ownership to ARC

## Code Highlights

Setup (define once before all includes):
```cpp
#define NS_PRIVATE_IMPLEMENTATION
#define CA_PRIVATE_IMPLEMENTATION
#define MTL_PRIVATE_IMPLEMENTATION
#include <Foundation/Foundation.hpp>
#include <Metal/Metal.hpp>
#include <QuartzCore/QuartzCore.hpp>
```

Draw a triangle in C++ with metal-cpp:
```cpp
MTL::CommandBuffer* pCmd = _pCommandQueue->commandBuffer();
MTL::RenderCommandEncoder* pEnc = pCmd->renderCommandEncoder(pRpd);
pEnc->setRenderPipelineState(_pPSO);
pEnc->drawPrimitives(MTL::PrimitiveTypeTriangle, NS::UInteger(0), NS::UInteger(3));
pEnc->endEncoding();
pCmd->presentDrawable(pView->currentDrawable());
pCmd->commit();
```

AutoreleasePool usage:
```cpp
NS::AutoreleasePool* pPool = NS::AutoreleasePool::alloc()->init();
MTL::CommandBuffer* pCmd = _pCommandQueue->commandBuffer(); // autoreleased
// ... use pCmd ...
pPool->release(); // drains pool, releasing pCmd etc.
```

NS::SharedPtr with TransferPtr:
```cpp
auto pDesc = NS::TransferPtr(MTL::RenderPipelineDescriptor::alloc()->init());
// pDesc.release() called automatically when out of scope
```

Loading a texture using MetalKit and bridging ownership:
```cpp
id<MTLTexture> texture = [textureLoader newTextureWithName:... ...];
return (__bridge_retained MTL::Texture*)texture; // take out of ARC
```

## Takeaways
- metal-cpp provides a zero-overhead, header-only C++ wrapper for the complete Metal API; the 1-to-1 mapping to Objective-C means Metal documentation, samples, and tooling apply directly.
- Cocoa's MRR conventions apply: own objects created with `alloc/new/copy/create`, release when done; use `NS::AutoreleasePool` on every thread to manage Metal-created autoreleased objects.
- `NS::TransferPtr` and `NS::RetainPtr` provide smart-pointer semantics for MRR objects, similar to ARC, eliminating manual `release` calls and reducing use-after-free bugs.
- Use the adapter pattern (Objective-C adapter calling C++, C++ adapter calling Objective-C) and `__bridge`/`__bridge_retained`/`__bridge_transfer` casts to cleanly integrate metal-cpp with other Objective-C frameworks.

---
_Source: WWDC22 Session 10160 page (abstract, chapter summaries, code samples, and resource links)._
