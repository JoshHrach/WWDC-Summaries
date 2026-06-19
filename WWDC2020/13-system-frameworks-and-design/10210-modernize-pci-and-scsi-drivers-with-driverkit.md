# Modernize PCI and SCSI drivers with DriverKit
**WWDC20 · Session 10210** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10210/)

_Platforms:_ macOS Big Sur 11

## Overview
DriverKit is Apple's user-space driver framework that replaces IOKit kernel extensions (kexts). Drivers built with DriverKit run as System Extensions in user space, providing better security (isolated resources, no kernel access), better reliability (crashes don't panic the kernel), and faster development iteration (no need for two machines or reboots).

macOS Big Sur adds two new DriverKit frameworks: **PCIDriverKit** for PCI/PCIe device drivers, and **SCSIControllerDriverKit** for SCSI controller drivers (Fibre Channel, SAS, RAID, etc.). Both are PCIe-based, and PCI/SCSI kexts that can be replaced by System Extensions are **deprecated** in macOS Catalina and will not load by default in a future macOS release. macOS Catalina was the last release with full kext support without compromises.

## Key Topics

### DriverKit System Extensions vs. Kernel Extensions
- Dexts (driver extensions) run in **user space**; isolated resources; crash without affecting the kernel
- Build, test, and debug on a single machine; no reboot on crash
- Installed via `SystemExtensions` framework (`OSSystemExtensionRequest.activationRequest`); stay inside the app bundle
- To test without App Store provisioning: disable SIP, sign locally, include all required entitlements
- Supported device families at WWDC20: USB, serial, NIC, HID, PCI, SCSI Controller

### PCIDriverKit
- Single class: `IOPCIDevice` — not subclassed; used as the **PCI provider** to access all PCI resources
- Requires a PCI entitlement in the app's entitlements file; matching uses IOPCIFamily criteria (vendor ID, device ID, with optional masking via `&`)
- **Exclusive access**: must call `Open(options:)` before any resource access; call `Close(options:)` in `Stop()`; PCI stack disables bus mastering and memory space after Close
- **Memory access**: framework manages all device memory mappings; use `ConfigurationRead/Write` and `MemoryRead/Write` APIs with `memoryIndex` + `offset`; memoryIndex ≠ BAR index for 64-bit BARs (two BARs = one 64-bit entry = one memory index)
- **Interrupt handling**: iterate interrupt indexes, find MSI/MSI-X type, create `IODispatchSource` with chosen dispatch queue, set `InterruptOccurred` as the handler
- **DMA**: create `IOBufferMemoryDescriptor`, create `IODMACommand` specifying the PCI device (ensures correct memory mapper), call `Prepare` to get physical address segments, pass physical address to hardware; call `Complete` after transfer

### SCSIControllerDriverKit
- New class: `IOUserSCSIParallelInterfaceController` — subclass this and implement pure-virtual functions
- API mirrors `IOSCSIParallelInterfaceController` (kext class) with "User" prefix (e.g., `UserProcessParallelTask`, `UserInitializeController`, `UserStartController`)
- Requires the SCSI family entitlement in addition to the generic DriverKit entitlement
- **Recommended DispatchQueue model**: three queues
  - Default Queue (provided by DriverKit) — handles all kernel-originated calls (I/O submission, `UserInitializeController`, etc.)
  - Interrupt Queue — separate queue for interrupt dispatch sources to prevent I/O and interrupt contention
  - Auxiliary Queue — used to create SCSI targets asynchronously (kernel makes re-entrant calls during target creation)
- **UserMapHBAData** — called by the kernel for each `SCSIParallelTask` the kernel allocates; create controller-specific task metadata here and pre-map buffers; never do this in the I/O path
- **I/O path**:
  - Kernel calls `UserProcessParallelTask(task: SCSIUserParallelTask, ...)` — task contains CDB, transfer count/direction, and `fBufferIOVMAddr` (physical start address of a single contiguous DMA segment already prepared by the kernel)
  - Build hardware-specific scatter-gather list if needed; post to hardware; stash completion callback
  - On interrupt: invoke `ParallelTaskCompletion(response: SCSIUserParallelResponse)` with completion status, bytes transferred, and optional sense data
- **Power management**: override `SetPowerState(powerState:)` — supports Off, Low-power, On; issue hardware reset in OnState; call `SUPERDISPATCH SetPowerState` after async initialization completes for deferred acknowledgement
- **Termination**: override `Stop()`; kernel handles outstanding I/O completions; close PCI session, cancel and release dispatch sources/queues in finalize block

### Deprecation Timeline
- macOS Catalina: kexts deprecated for USB, serial, NIC, HID (DriverKit supported)
- macOS Big Sur: kexts deprecated for PCI and SCSI Controller (new frameworks); deprecated kexts from Catalina **do not load by default**
- Future macOS: PCI and SCSI kexts will not load by default

## APIs & Frameworks

- **PCIDriverKit** **[NEW]**
  - `IOPCIDevice` **[NEW]** — PCI provider; not subclassed; call `Open(options:)` before any access, `Close(options:)` at stop
  - `IOPCIDevice.ConfigurationRead8/16/32(offset:)` / `ConfigurationWrite8/16/32(offset:data:)` **[NEW]** — PCI config space access
  - `IOPCIDevice.MemoryRead8/16/32/64(memoryIndex:offset:)` / `MemoryWrite…` **[NEW]** — MMIO device register access
  - `IOPCIDevice.FindInterruptType(index:interruptType:)` **[NEW]** — iterate interrupt indexes to find MSI/MSI-X
  - `IODispatchSource.CreateWithInterrupt(device:index:dispatchQueue:handler:source:)` **[NEW]** — interrupt dispatch source
  - `IOBufferMemoryDescriptor` — buffer memory descriptor for DMA
  - `IODMACommand` **[NEW]** — DMA command; specifies PCI device to select correct memory mapper
  - `IODMACommand.PrepareForDMA(...)` **[NEW]** — prepares buffer and returns physical segment(s)
  - `IODMACommand.CompleteDMA(...)` **[NEW]** — releases physical memory after transfer
- **SCSIControllerDriverKit** **[NEW]**
  - `IOUserSCSIParallelInterfaceController` **[NEW]** — base class; subclass and implement pure-virtual functions
  - `UserInitializeController(...)` **[NEW]** — set up queues, interrupt sources, PCI session
  - `UserStartController(...)` **[NEW]** — enable hardware interrupts; return Success when ready for I/O
  - `UserMapHBAData(task:taskData:)` **[NEW]** — allocate per-task controller metadata; called at task allocation time
  - `UserProcessParallelTask(task:response:)` **[NEW]** — submit I/O to hardware; `task.fBufferIOVMAddr` is the DMA physical address
  - `SCSIUserParallelTask` **[NEW]** — task struct: CDB, data transfer count/direction, `fBufferIOVMAddr`
  - `SCSIUserParallelResponse` **[NEW]** — completion struct: status, bytes transferred, sense data
  - `ParallelTaskCompletion(response:)` **[NEW]** — OS action callback to signal I/O completion to kernel
  - `SetPowerState(powerState:)` **[NEW]** — power management; `kIOSCSIPowerStateOff`, `kIOSCSIPowerStateOn`
- **SystemExtensions**
  - `OSSystemExtensionRequest.activationRequest(forExtensionWithIdentifier:queue:)` — install/activate the dext from the app
- **Entitlements** (new for PCI/SCSI)
  - `com.apple.developer.driverkit.transport.pci` — PCI device family entitlement; takes array of IOPCIFamily matching dictionaries
  - `com.apple.developer.driverkit.family.scsicontroller` — SCSI controller family entitlement **[NEW]**

## Code Highlights

Enabling bus mastering and memory space at PCI Start:
```cpp
kern_return_t MyDriver::Start_Impl(IOService* provider) {
    ivars->pciDevice->Open(this, 0);
    uint16_t cmd = 0;
    ivars->pciDevice->ConfigurationRead16(kIOPCIConfigurationOffsetCommand, &cmd);
    cmd |= kIOPCICommandBusMaster | kIOPCICommandMemorySpace;
    ivars->pciDevice->ConfigurationWrite16(kIOPCIConfigurationOffsetCommand, cmd);
    // ... rest of setup
}
```

Setting up a DMA command with physical address retrieval:
```cpp
IOBufferMemoryDescriptor* buf = nullptr;
IOBufferMemoryDescriptor::Create(kIOMemoryDirectionInOut, length, 0, &buf);

IODMACommand* dmaCmd = nullptr;
IODMACommand::Create(ivars->pciDevice, 0, &dmaSpec, &dmaCmd);

uint64_t physAddr = 0; uint64_t segLen = length;
dmaCmd->PrepareForDMA(kIODMACommandPrepareForDMANoOptions, buf, 0, length, &flags, &physAddr, &segLen);
// Write physAddr to hardware DMA register
// ... after transfer:
dmaCmd->CompleteDMA(kIODMACommandCompleteDMANoOptions);
```

Processing a SCSI parallel task:
```cpp
kern_return_t ExampleSCSIDext::UserProcessParallelTask_Impl(
    SCSIUserParallelTask task, SCSIUserParallelResponse* response) {
    uint64_t physAddr = task.fBufferIOVMAddr;
    // Build SGL from physAddr + task.fTransferCount
    // Post to hardware
    return kIOReturnSuccess;
}
```

## Takeaways
- PCI and SCSI controller kexts are deprecated in macOS Big Sur and will not load by default in a future release — migrate to `PCIDriverKit` and `SCSIControllerDriverKit` now.
- `IOPCIDevice` must be `Open`'d before any PCI resource access and `Close`'d at `Stop()`; the framework, not the driver, manages device memory mappings.
- For SCSI dexts, the kernel prepares all DMA segments and passes a single physical address in `fBufferIOVMAddr` — never prepare `IODMACommand` in the I/O path; pre-process and pre-map everything in `UserMapHBAData`.
- Use three DispatchQueues (Default, Interrupt, Auxiliary) to avoid I/O and interrupt contention and to allow safe asynchronous SCSI target creation.
- SCSIControllerDriverKit achieves equivalent throughput to kexts despite the user-space/kernel IPC boundary; demonstrated at ~350 MB/s with a 4 Gb/s Fibre Channel controller.

---
_Source: WWDC20 Session 10210 page (abstract and transcript)._
