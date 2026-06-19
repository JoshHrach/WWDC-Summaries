# Speedrun Your Game Port with Agentic Coding
**WWDC26 · Session 357** · [Watch](https://developer.apple.com/videos/play/wwdc2026/357/)

_Platforms:_ macOS, iOS, iPadOS

## Overview
This session introduces Game Porting Toolkit 4, which adds a suite of agentic skills that give an AI coding assistant expert domain knowledge about porting games to Apple platforms. Instead of requiring developers to manually look up Metal 4 APIs, shader converter conventions, and MetalFX integration patterns, the agent can autonomously complete structured porting milestones—including GPU debugging—while the developer focuses on high-level decisions.

The porting assistant operates in three phases: Discover (scanning the codebase and capturing reference frames from the Game Porting Toolkit evaluation environment), Plan (collaboratively defining milestones), and Execute-and-Validate (running the agent against structured checklists that verify app launch, Metal API correctness, screen captures, anti-pattern detection, and memory issues). This structured validation loop ensures each milestone is provably correct before the agent moves forward.

The session covers the full porting pipeline from windowing and frame pacing through shader pipeline migration, GPU debugging, game controller input, and MetalFX integration, with each area having dedicated expert skills the agent can invoke. New GPU command-line tools in macOS 27 (`gpucapture` and `gpudebug`) enable fully autonomous GPU debugging without Xcode interaction.

## Key Topics

### Porting Assistant Workflow (1:06)
Skills are installed into the coding agent from the GPTK 4 GitHub repository using plugin marketplace commands. Two skill types are provided: expert skills (deep API knowledge for specific subsystems) and workflow skills (orchestration of multi-step tasks across expert skills). The porting assistant workflow skill coordinates the full Discover → Plan → Execute-and-Validate loop, with the validation checklist including app launch verification, Metal API correctness checks, screen capture comparison against reference frames, anti-pattern review, and memory issue detection.

### Windowing and Frame Pacing (6:25)
The window, Metal swap chain, presenting drawables, and metal-cpp expert skills guide the agent to implement a correct render loop. Key patterns covered: using Metal Display Link for game-loop pacing, configuring the `CAMetalLayer` correctly for games (pre-warming drawables, setting `displaySyncEnabled`), using direct-to-display presentation for reduced latency, and correct drawable lifetime management to prevent stuttering from stale drawables.

### Scene Rendering with Metal 4 (8:28)
Three sub-milestones are addressed by dedicated expert skills:
1. **GPU resource management** — storage modes, residency sets, constant buffer allocation strategy
2. **Shader pipeline setup** — Metal Shader Converter argument buffer offsets via `IRRootSignatureGetResourceLocations`, descriptor table encoding, root signature parameter count from shader reflection via `IRShaderReflectionCreate`
3. **Command encoding and synchronization** — mapping D3D12 resource states to Metal 4 producer-consumer barrier stages using `MTL4::VisibilityOptionDevice`

### Autonomous GPU Debugging (15:16)
New `gpucapture` and `gpudebug` command-line tools on macOS 27 allow an agent to capture a GPU frame trace, inspect pipeline state, texture contents, and dispatch dimensions—entirely from the terminal. This enables the agent to identify and fix rendering artifacts (e.g., a misbound texture causing incorrect lighting) without requiring a developer to manually operate Xcode's GPU frame debugger.

### Game Controllers and MetalFX (19:09)
The game controller expert skill ports Windows XInput calls to `GCController` with correct device discovery, dynamic button layout querying, and connect/disconnect event handling. The MetalFX upscaling and frame interpolation skills handle jitter configuration, motion vector conventions, mip bias, history reprojection, and dedicated present thread setup. New Metal HUD debugging overlays in macOS 27 allow in-situ validation of MetalFX integration quality.

## APIs & Frameworks

### Game Porting Toolkit 4 (NEW)
- **[NEW]** Game Porting Toolkit 4 agentic skills — installable plugin skills for AI coding agents
- **[NEW]** Porting assistant workflow skill — orchestrates Discover, Plan, Execute-and-Validate phases
- Expert skills: window, Metal swap chain, presenting drawables, metal-cpp, GPU resources, shader pipeline, Metal Shader Converter, synchronization, game controller, MetalFX upscaling, MetalFX frame interpolation
- **[NEW]** `/plugin marketplace add apple/game-porting-toolkit` — CLI command to add GPTK skill source
- **[NEW]** `/plugin install game-porting-skills@game-porting-toolkit` — CLI command to install skills

### Metal 4
- `MTL4::VisibilityOptionDevice` — device-level visibility scope for explicit barriers
- `barrierAfterStages(_:producerStages:consumerStages:visibility:)` — explicit synchronization barrier
- `MTL::StageDispatch`, `MTL::StageAll` — pipeline stage tokens
- Residency sets (`MTLResidencySet`) — explicit GPU resource residency management
- `addAllocation(_:)` / `commit()` — residency set population and finalization
- `gpuAddress` — GPU virtual address for bindless resource access
- Argument tables / `setAddress(_:atIndex:)` — bindless resource binding

### Metal Shader Converter / HLSL-to-Metal
- `IRRootSignatureGetResourceLocations(_:_:)` — query argument buffer offsets from root signature reflection
- `IRShaderReflectionCreate()` — create a shader reflection object
- `IRObjectGetReflection(_:_:_:)` — populate reflection for a compiled IR object
- `IRShaderStageCompute` — shader stage token for compute reflection queries
- `IRRootSignature` / `Reset(_:_:)` — root signature construction

### Metal Display Link
- `CAMetalDisplayLink` — display-sync callback for game render loops
- `CAMetalLayer` — Metal swap chain layer; `displaySyncEnabled`, drawable pre-warming

### GCController / Game Controller Framework
- `GCController` — unified game controller type
- `GCExtendedGamepad` — extended gamepad profile
- `GCControllerButtonInput` — button input element
- Device discovery, `GCControllerDidConnectNotification`, `GCControllerDidDisconnectNotification`
- Dynamic button layout querying (replaces hardcoded XInput button indices)

### MetalFX
- MetalFX Upscaling (`MTLFXTemporalScaler`) — temporal super-resolution
- MetalFX Frame Interpolation — frame rate doubling via AI-driven interpolation
- Jitter configuration, motion vector conventions, mip bias, history reprojection
- Dedicated present thread pattern for frame interpolation
- **[NEW]** Metal HUD MetalFX debugging overlays (macOS 27)

### GPU Debugging Tools (NEW, macOS 27)
- **[NEW]** `gpucapture` CLI — command-line GPU frame capture
- **[NEW]** `gpudebug` CLI — command-line GPU frame inspection (pipeline state, textures, dispatches)

## Code Highlights

Installing GPTK 4 agentic skills:
```
/plugin marketplace add apple/game-porting-toolkit
/plugin install game-porting-skills@game-porting-toolkit
```

Using residency sets for Metal 4 resource management:
```cpp
residencySet->addAllocation(texture);
residencySet->commit();
argumentTable->setAddress(texture->gpuAddress(), bindPoint);
```

Querying argument buffer offsets from root signature reflection:
```cpp
IRRootSignatureGetResourceLocations(m_MtlCurIRRootSig, locations);
size_t offset = locations[i].topLevelOffset;
```

Mapping D3D12 states to Metal 4 producer-consumer barriers:
```cpp
m_MtlPendingProducerStages |= MtlProducerStageFromD3D12(OldState);
m_MtlPendingConsumerStages |= MtlConsumerStageFromD3D12(NewState);
m_ComputeEncoder->barrierAfterStages(
    m_MtlPendingProducerStages,
    m_MtlPendingConsumerStages,
    MTL4::VisibilityOptionDevice);
```

## Takeaways
- Game Porting Toolkit 4 agentic skills give an AI coding assistant enough expert knowledge to complete structured porting milestones autonomously, dramatically reducing the manual research burden.
- The Discover → Plan → Execute-and-Validate loop with structured checklists keeps the agent on track and prevents silent regressions.
- New `gpucapture` / `gpudebug` CLI tools enable fully autonomous GPU debugging without Xcode, which is the key enabler for an agent-driven porting workflow.
- MetalFX frame interpolation and upscaling have dedicated skills that handle the many subtle configuration details (jitter, motion vectors, mip bias, present thread) that are easy to get wrong manually.

---
_Source: WWDC26 Session 357 page (abstract, chapter summaries, and code samples)._
