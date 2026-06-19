# Explore the New System Architecture of Apple Silicon Macs
**WWDC20 · Session 10686** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10686/)

_Platforms:_ macOS Big Sur 11

## Overview
This session introduces the hardware and software architecture of the first Macs with Apple silicon, covering the System-on-Chip (SoC) design, security enhancements, application compatibility via Rosetta 2, and the new boot/recovery architecture. The unified memory architecture means the CPU and GPU share the same memory pool, eliminating PCIe copy overhead for graphics resources. Apple silicon also brings the Neural Engine, hardware video codecs, and asymmetric multiprocessing (AMP) to Mac for the first time.

Security capabilities from iPhone come to Mac: Write XOR Execute enforcement, Kernel Integrity Protection, Pointer Authentication Codes, and per-device IOMMU memory isolation. On the boot side, a new Secure Boot chain (derived from iOS/iPadOS) and a redesigned macOS Recovery with per-volume security policies replace the Intel approach.

Rosetta 2 transparently translates x86_64 binaries — including apps with JIT compilers — by pre-translating at install time and JIT-compiling new code at runtime.

## Key Topics

**Unified Memory Architecture**
- Single SoC combines CPU, GPU, Neural Engine, video codecs, and ML accelerators
- CPU and GPU share the same physical memory — no PCIe copy for textures, images, or geometry
- Same Metal API on both Intel and Apple silicon; Apple silicon delivers significant throughput gains

**Asymmetric Multiprocessing (AMP)**
- Mix of high-performance and power-efficient CPU cores
- macOS schedules tasks across all cores based on Quality of Service (QoS)
- Correct QoS assignment and Grand Central Dispatch usage are essential for optimal scheduling
- `DispatchQueue.concurrentPerform` helps load-balance across heterogeneous cores

**Hardware Accelerators**
- Neural Engine accelerates Core ML — no code changes needed; ensure `computeUnits` is `.all` (the default), not `.cpuOnly` or `.cpuAndGPU`
- Accelerate framework has tuned implementations for Apple silicon
- BiPlanar pixel formats (e.g., `kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange`) are preferred for hardware video codec efficiency via AVFoundation and VideoToolbox

**Security Enhancements**
- Write XOR Execute: memory pages cannot be simultaneously writable and executable; new per-thread API allows fast toggling for JIT compilers
- Kernel Integrity Protection: hardware-enforced immutability of kernel code pages after boot
- Pointer Authentication Codes (PAC): unused bits in 64-bit pointers store a cryptographic code checked at pointer use
- Device Isolation: each PCIe device gets a separate IOMMU memory mapping; drivers must use `IOMapper` + `IODMACommand` API (not raw `getPhysicalSegment`)
- Kernel extensions now require a reboot to load; PAC must be adopted in kexts; DriverKit recommended as replacement

**Per-Volume Security Policy**
- Full Security (default): same security as iPhone; secure external boot supported without downgrading
- Reduced Security: allows unsigned/older macOS, notarized kexts; requires authentication in Recovery
- `CSRUtil` tool configures fine-grained security for developers and researchers

**Boot and Recovery Architecture**
- Secure Boot chain from iOS/iPadOS: each component cryptographically signed by Apple
- Hold TouchID/Power button → Startup Options (replaces separate startup key combos)
- Mac Sharing Mode replaces Target Disk Mode; uses SMB with user authentication
- System Recovery: hidden container with minimal macOS for restoring when main Recovery is damaged
- Apple Configurator 2: recovers Mac when System Recovery itself is non-functional
- Unified Login experience: full macOS boots before unlock; supports smart cards (CCID/PIV), VoiceOver, FileVault

**Data Protection**
- Full data volume encryption enabled by default (like T2 Macs)
- FileVault ties encryption to user credentials
- Secure Hibernation: integrity and anti-replay protection for in-memory state during low-battery events

**Rosetta 2**
- Translates x86_64 binaries at install time (triggered by App Store or package installer)
- JIT-compiles any untranslated code at launch
- Fully emulates x86_64 process including system call interface and all system frameworks
- Translated apps can use Metal (generates native GPU commands) and Core ML (runs on Neural Engine)
- Does NOT support AVX vector extensions; apps must check for AVX before use
- `sysctl.proc_translated` detects Rosetta environment at runtime
- Differences from native: page size, memory ordering, `mach_absolute_time` frequency, floating-point edge cases

## APIs & Frameworks

### Core ML
- `MLComputeUnits.all` — default; enables Neural Engine on Apple silicon
- `MLComputeUnits.cpuOnly` / `.cpuAndGPU` — bypass Neural Engine; avoid unless required

### Grand Central Dispatch
- `DispatchQueue.concurrentPerform(iterations:execute:)` — parallel task distribution across AMP cores
- `DispatchQoS` — Quality of Service levels: `.userInteractive`, `.userInitiated`, `.utility`, `.background`; critical on AMP systems

### Metal
- Same API on Intel and Apple silicon; GPU accesses unified memory directly with no copy overhead

### AVFoundation / VideoToolbox
- BiPlanar pixel formats preferred for hardware codec efficiency on Apple silicon (e.g., `kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange`)

### IOKit (kernel extensions)
- `IOMapper::copyMapperForDevice(device)` — get the IOMMU mapper for a PCIe device
- `IODMACommand::withSpecification(outSegFunc:numAddressBits:maxSegmentSize:mappingOptions:maxTransferSize:alignment:mapper:refCon:)` — set up DMA with device-specific mapper
- `getPhysicalSegment` (deprecated path) — does not work under per-device IOMMU on Apple silicon

### System Calls / sysctl
- `sysctlbyname("sysctl.proc_translated", ...)` **[NEW]** — returns 1 if process is running under Rosetta

## Code Highlights

Checking if running under Rosetta:
```c
int processIsTranslated() {
    int ret = 0;
    size_t size = sizeof(ret);
    if (sysctlbyname("sysctl.proc_translated", &ret, &size, NULL, 0) != -1)
        return ret;
    if (errno == ENOENT) return 0;  // native — sysctl not present
    return -1;
}
```

Correct IOKit DMA setup for Apple silicon:
```cpp
IOMapper *mapper = IOMapper::copyMapperForDevice(device);
IODMACommand *dmaCommand = IODMACommand::withSpecification(
    outSegFunc, numAddressBits, maxSegmentSize,
    mappingOptions, maxTransferSize, alignment, mapper, refCon);
// Keep prepared for duration of I/O
```

## Takeaways
- No new APIs are required for most app improvements on Apple silicon — use Metal, Core ML (with `computeUnits = .all`), AVFoundation BiPlanar formats, Accelerate, and GCD with correct QoS.
- Kernel extension developers must adopt `IOMapper`/`IODMACommand` for DMA and prepare for PAC requirements; migrate to DriverKit when possible.
- Each macOS installation on Apple silicon can have its own security policy (Full or Reduced), enabling a dedicated development volume without compromising the primary install.
- Rosetta 2 handles most x86_64 apps transparently but does not translate AVX instructions; apps must guard AVX use with a capability check.

---
_Source: WWDC20 Session 10686 page (abstract, chapter summaries, code samples, and resource links)._
