# Create Audio Drivers with DriverKit
**WWDC21 · Session 10190** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10190/)

_Platforms:_ macOS Monterey 12

## Overview
macOS Monterey introduces AudioDriverKit, a new DriverKit framework that replaces the previous two-component audio driver model (Audio Server plug-in + DriverKit Extension) with a single Driver Extension (dext). This eliminates inter-process communication between the plug-in and dext, simplifies development, reduces overhead and latency, and allows distribution through the Mac App Store with no separate installer package. Drivers load immediately without a reboot.

The session covers the complete AudioDriverKit programming model: architecture, required entitlements, dext configuration, creating `IOUserAudioDevice` and `IOUserAudioStream` objects, handling the IO path with precise zero-timestamp tracking, and performing device configuration changes (sample rate switching) via the `RequestDeviceConfigurationChange` / `PerformDeviceConfigurationChange` pair.

## Key Topics

### Architecture
`AudioDriverKit` handles all IPC between the dext and Core Audio HAL by creating a private `IOUserAudioDriverUserClient` automatically. The dext does not need to implement or open this user client directly. Optionally, a custom user client (for app-to-dext communication) can be created by overriding `NewUserClient` and calling `IOService::Create`. The Audio Server plug-in model remains supported and is not deprecated; virtual audio drivers should continue using it.

### Entitlements Required
- DriverKit entitlement (all dexts)
- `com.apple.developer.driverkit.allow-any-userclient-access` (required for AudioDriverKit)
- Transport family entitlements as needed (USB, PCI)

### Class Hierarchy
- `IOUserAudioDriver : IOService` — top-level driver; override `Start`, `Stop`, `NewUserClient`, `StartDevice`, `StopDevice`
- `IOUserAudioDevice : IOUserAudioClockDevice` — represents one audio device; owns streams and controls; override `StartIO`, `StopIO`, `PerformDeviceConfigurationChange`
- `IOUserAudioStream` — owns an `IOBufferMemoryDescriptor` for the audio ring buffer; mapped to Core Audio HAL
- `IOUserAudioLevelControl` — volume control (dB or scalar value)
- `IOUserAudioCustomProperty` — arbitrary key-value property attached to any `IOUserAudioObject`

### IO Path and Zero Timestamps
`IOUserAudioClockDevice` provides `UpdateCurrentZeroTimestamp(sampleTime, hostTime)` and `GetCurrentZeroTimestamp`. The HAL uses the sample-time/host-time pair to schedule and synchronize IO. The dext must track the hardware clock's timestamps as precisely as possible using an `IOTimerDispatchSource` that fires at each buffer period. On each timer fire: retrieve the previous timestamp, advance by `GetZeroTimestampPeriod()` (samples) and `hostTicksPerBuffer`, call `UpdateCurrentZeroTimestamp`, then reschedule the timer one buffer ahead.

Audio data is written to the `IOMemoryMap` obtained from the stream's `IOMemoryDescriptor` (simulating DMA). In the sample, a sine wave is generated and written to a signed 16-bit PCM ring buffer.

### Device Configuration Changes
For any change to IO-affecting state (sample rate, stream format), the driver must:
1. Call `RequestDeviceConfigurationChange(action, changeInfo)` — tells the HAL to stop IO and call back.
2. HAL stops IO, then calls `PerformDeviceConfigurationChange(action, changeInfo)` on the driver.
3. Inside `PerformDeviceConfigurationChange`, update `SetSampleRate` and call `DeviceSampleRateChanged` on the stream.
Only within `PerformDeviceConfigurationChange` is it safe to change IO-affecting state.

## APIs & Frameworks

**AudioDriverKit** (C++ DriverKit framework) — **[NEW macOS 12]**

- `IOUserAudioDriver : IOService` **[NEW]** — base driver class
  - `StartDevice(objectID:flags:)` / `StopDevice(objectID:flags:)` — HAL IO start/stop
  - `NewUserClient(inType:outUserClient:)` — create user clients
- `IOUserAudioDevice : IOUserAudioClockDevice` **[NEW]** — audio device object
  - `SetAvailableSampleRates(_:count:)` / `SetSampleRate(_:)` — configure sample rates
  - `AddStream(_:)` / `AddControl(_:)` / `AddCustomProperty(_:)` — add child objects
  - `StartIO(flags:)` / `StopIO(flags:)` — IO lifecycle
  - `RequestDeviceConfigurationChange(action:changeInfo:)` — initiate config change
  - `PerformDeviceConfigurationChange(action:changeInfo:)` — apply config change (override)
- `IOUserAudioClockDevice` **[NEW]** — base class providing timestamp API
  - `UpdateCurrentZeroTimestamp(sampleTime:hostTime:)` — atomically update clock
  - `GetCurrentZeroTimestamp(sampleTime:hostTime:)` — read current clock
  - `GetZeroTimestampPeriod()` — samples per buffer (anchor period)
- `IOUserAudioStream` **[NEW]** — audio stream object
  - `Create(driver:direction:ioMemoryDescriptor:)` — factory method
  - `SetAvailableStreamFormats(_:count:)` / `SetCurrentStreamFormat(_:)` — format config
  - `GetIOMemoryDescriptor()` — retrieve ring buffer descriptor
  - `DeviceSampleRateChanged(_:)` — update stream format on sample rate change
  - `IOUserAudioStreamDirection` — `.Input` / `.Output`
  - `IOUserAudioStreamBasicDescription` — format struct (sampleRate, formatID, flags, bytesPerFrame, channelsPerFrame, bitsPerChannel)
- `IOUserAudioLevelControl` **[NEW]** — volume control
  - `Create(driver:isSettable:initialLevel:levelRange:element:scope:class:)` — factory
  - `GetScalarValue()` — read current volume as 0.0–1.0
- `IOUserAudioCustomProperty` **[NEW]** — arbitrary property on any audio object
  - `Create(driver:propertyAddress:isSettable:qualifierType:dataType:)` — factory
  - `SetQualifierAndDataValue(_:_:)` — set property data
- `IOUserAudioObjectPropertyAddress` — `{selector, scope, element}` struct
- `IOUserAudioDriverUserClient` **[NEW]** — private HAL user client; created automatically by base class

**DriverKit** (`IOTimerDispatchSource`, `IOBufferMemoryDescriptor`, `IOMemoryMap`, `OSAction`)
- `IOTimerDispatchSource::WakeAtTime(clock:time:leeway:)` — schedule timer
- `IOBufferMemoryDescriptor::Create(direction:capacity:alignment:descriptor:)` — allocate ring buffer
- `IOMemoryDescriptor::CreateMapping(...)` — map memory for CPU access

## Code Highlights

Driver Start — create and register device:
```cpp
kern_return_t SimpleAudioDriver::Start_Impl(IOService* provider) {
    super::Start(provider, SUPERDISPATCH);
    ivars->m_simple_audio_device = OSSharedPtr(
        OSTypeAlloc(SimpleAudioDevice), OSNoRetain);
    ivars->m_simple_audio_device->init(this, ...);
    AddObject(ivars->m_simple_audio_device.get());
    RegisterService();
    return kIOReturnSuccess;
}
```

StartIO — map ring buffer, start timestamp timer:
```cpp
kern_return_t SimpleAudioDevice::StartIO(IOUserAudioStartStopFlags in_flags) {
    super::StartIO(in_flags);
    input_iomd = ivars->m_input_stream->GetIOMemoryDescriptor();
    input_iomd->CreateMapping(0, 0, 0, 0, 0, ivars->m_input_memory_map.attach());
    StartTimers();
    return kIOReturnSuccess;
}
```

Zero timestamp update in timer callback:
```cpp
void SimpleAudioDevice::ZtsTimerOccurred_Impl(OSAction* action, uint64_t time) {
    GetCurrentZeroTimestamp(&current_sample_time, &current_host_time);
    if (current_host_time != 0) {
        current_sample_time += GetZeroTimestampPeriod();
        current_host_time += ivars->m_zts_host_ticks_per_buffer;
    } else {
        current_sample_time = 0;
        current_host_time = time;
    }
    UpdateCurrentZeroTimestamp(current_sample_time, current_host_time);
    ivars->m_zts_timer_event_source->WakeAtTime(
        kIOTimerClockMachAbsoluteTime,
        current_host_time + ivars->m_zts_host_ticks_per_buffer, 0);
}
```

Sample rate config change:
```cpp
kern_return_t SimpleAudioDevice::PerformDeviceConfigurationChange(
    uint64_t change_action, OSObject* in_change_info) {
    double new_rate = (GetSampleRate() != kSampleRate_1) ? kSampleRate_1 : kSampleRate_2;
    SetSampleRate(new_rate);
    ivars->m_input_stream->DeviceSampleRateChanged(new_rate);
    return kIOReturnSuccess;
}
```

## Takeaways
- AudioDriverKit consolidates Audio Server plug-in + DriverKit Extension into a single dext, eliminating inter-process overhead and enabling Mac App Store distribution without an installer.
- The `IOUserAudioDriverUserClient` for HAL communication is created automatically; dexts call `super::NewUserClient` for HAL connections and `IOService::Create` for custom app connections.
- Precise zero-timestamp tracking via `UpdateCurrentZeroTimestamp` is critical — use an `IOTimerDispatchSource` firing once per buffer period, advancing sample and host times by fixed periods.
- Device configuration changes (sample rate, format) must go through the `RequestDeviceConfigurationChange` / `PerformDeviceConfigurationChange` handshake so the HAL can safely stop and restart IO.
- The Audio Server plug-in model remains supported; AudioDriverKit is for hardware devices with DriverKit-supported transport families (USB, PCI), not virtual devices.

---
_Source: WWDC21 Session 10190 page (abstract, chapter summaries, code samples, and resource links)._
