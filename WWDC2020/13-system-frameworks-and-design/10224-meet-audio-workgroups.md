# Meet Audio Workgroups
**WWDC20 · Session 10224** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10224/)

_Platforms:_ macOS Big Sur 11, iOS 14

## Overview
Audio workgroups are a new kernel-level scheduling primitive introduced in macOS Big Sur and iOS 14 that allow audio applications and Audio Unit plug-ins to register their realtime threads with the system performance controller. Modern CPUs (especially Apple silicon SoC) dynamically trade off power and performance; by joining realtime audio threads to workgroups, the kernel understands which threads share a deadline and can schedule them appropriately to avoid audio glitches.

The core audio server's own I/O threads are automatically managed by the system. The new APIs are needed only for apps that create their own realtime audio threads. Audio Unit developers who create auxiliary rendering threads must implement the new `AURenderContextObserver` property so the host can communicate which workgroup the plug-in should join at render time.

## Key Topics

### What is an Audio Workgroup?
- A workgroup consists of one or more cooperating realtime threads sharing a common deadline
- The master thread signals the kernel at the start (`os_workgroup_interval_start`) and end (`os_workgroup_interval_finish`) of each work cycle, providing a start time and deadline
- Other threads join the workgroup with `os_workgroup_join` (once per thread lifetime) and leave with `os_workgroup_leave`
- The kernel uses this information to make better scheduling decisions across all workgroup threads

### What the System Handles Automatically
- Core Audio's I/O server thread is the master of a device workgroup
- The client-side render thread in an audio application is automatically joined to the device workgroup by AVAudioEngine, AUGraph, and related frameworks
- Remote Audio Unit hosting threads (inter-process rendering) are also automatically joined by the system

### App Realtime Threads Parallel to the I/O Thread
- When an app creates auxiliary threads that do parallel audio work with the same deadline as the I/O thread, join them to the device's workgroup
- Obtain the device workgroup on macOS via the HAL API on the audio device
- On any platform, obtain via a property on the I/O Audio Unit (`AUHAL` on macOS, `AURemoteIO` on iOS)
- Call `os_workgroup_join` in each realtime thread once on startup; call `os_workgroup_leave` before the thread exits

### App Realtime Threads with Independent Deadlines
- When auxiliary threads have a different cadence/deadline from the device I/O thread, create an independent workgroup with `AudioWorkIntervalCreate`
- Join threads to this custom workgroup with `os_workgroup_join`
- The master thread calls `os_workgroup_interval_start(startTime, deadline, ...)` at the beginning of each cycle; `startTime` is `mach_absolute_time()` or the actual scheduled start; deadline must exceed start time
- Call `os_workgroup_interval_finish` at the end of each cycle
- Use `mach_timebase_info` to convert `mach_absolute_time` units — do not assume nanoseconds

### Audio Unit Realtime Threads
- Audio Units with auxiliary rendering threads don't know which thread context they will be called in (device I/O thread, custom workgroup, etc.)
- Implement the `AURenderContextObserver` property **[NEW]**: a block called before each render request
- The block receives an `AudioUnitRenderContext` struct containing an `os_workgroup`
- When the workgroup changes, join auxiliary threads to the new workgroup via `os_workgroup_join`; leave the old workgroup first
- The workgroup may change from render call to render call; always check for changes
- The system calls `AURenderContextObserver` automatically — the host must NOT call it manually

### Thread Count Recommendation
- `os_workgroup_max_parallel_threads` **[NEW]** — returns the recommended maximum number of parallel threads for a given workgroup
- For Audio Units: create threads equal to the CPU core count at initialization time; at render time, use only `os_workgroup_max_parallel_threads` threads

## APIs & Frameworks

- **os (libkern / os_workgroup.h)**
  - `os_workgroup_t` — opaque workgroup handle
  - `os_workgroup_join(_:_:)` **[NEW]** — joins the calling thread to a workgroup; call once per thread lifetime
  - `os_workgroup_leave(_:_:)` **[NEW]** — removes the calling thread from a workgroup; call before thread exits
  - `os_workgroup_interval_start(_:_:_:_:)` **[NEW]** — signals the start of a work cycle on the master thread; takes `startTime` and `deadline` in `mach_absolute_time` units
  - `os_workgroup_interval_finish(_:_:)` **[NEW]** — signals the end of a work cycle on the master thread
  - `os_workgroup_max_parallel_threads(_:_:)` **[NEW]** — recommended max parallel threads for a workgroup
- **Core Audio (HAL / AudioToolbox)**
  - `AudioWorkIntervalCreate(_:_:_:)` **[NEW]** — creates a custom audio workgroup with a specified interval; returns an `os_workgroup_t`
  - HAL property for device workgroup (macOS): obtain `os_workgroup_t` from the audio device object via `AudioObjectGetPropertyData`
  - `AUHAL` / `AURemoteIO` property for device workgroup: `kAudioOutputUnitProperty_OSWorkgroup` — returns the device's `os_workgroup_t`
- **Audio Unit (AudioToolbox)**
  - `AURenderContextObserver` **[NEW]** — property on an Audio Unit; value is a block `(AudioUnitRenderContext) -> Void`
  - `AudioUnitRenderContext` **[NEW]** — struct containing an `os_workgroup` field; passed to the `AURenderContextObserver` block before each render
- **mach / libSystem**
  - `mach_absolute_time()` — monotonic clock for workgroup timestamps
  - `mach_timebase_info(_:)` — get clock frequency to convert `mach_absolute_time` to nanoseconds

## Code Highlights

Joining an app's parallel threads to the device workgroup (iOS):
```c
// Get device workgroup from AURemoteIO
os_workgroup_t workgroup = NULL;
UInt32 size = sizeof(workgroup);
AudioUnitGetProperty(ioUnit,
    kAudioOutputUnitProperty_OSWorkgroup,
    kAudioUnitScope_Global, 0,
    &workgroup, &size);

// In each realtime thread (once):
os_workgroup_join_result_t result = os_workgroup_join(workgroup, &token);
```

Creating an independent workgroup for asynchronous realtime threads:
```c
os_workgroup_t myWorkgroup = NULL;
AudioWorkIntervalCreate("MyWorkgroup", OS_CLOCK_MACH_ABSOLUTE_TIME, &myWorkgroup);

// Per-cycle in the master thread:
os_workgroup_interval_start(myWorkgroup, mach_absolute_time(), deadline, &params);
// ... do work ...
os_workgroup_interval_finish(myWorkgroup, &params);
```

Audio Unit render context observer (join to host's workgroup):
```objc
// In your AUAudioUnit subclass:
self.renderContextObserver = ^(const AURenderContext *context) {
    if (context->workgroup != currentWorkgroup) {
        os_workgroup_leave(currentWorkgroup, &oldToken);
        os_workgroup_join(context->workgroup, &newToken);
        currentWorkgroup = context->workgroup;
    }
};
```

## Takeaways
- All realtime audio threads must be joined to a workgroup so the performance controller can schedule them correctly relative to their deadline — threads not in a workgroup may miss deadlines and cause glitches.
- The system handles its own threads automatically; apps only need to act if they create their own realtime threads.
- Use `os_workgroup_join` with the device's workgroup for threads parallel to the I/O cycle; use `AudioWorkIntervalCreate` + `os_workgroup_interval_start/finish` for threads with independent deadlines.
- Audio Unit plug-ins with auxiliary rendering threads must implement `AURenderContextObserver` — the only correct way to discover which workgroup to join at render time.

---
_Source: WWDC20 Session 10224 page (abstract and transcript)._
