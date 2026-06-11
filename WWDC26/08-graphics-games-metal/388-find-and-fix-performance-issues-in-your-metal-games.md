# Find and Fix Performance Issues in Your Metal Games
**WWDC26 · Session 388** · [Watch](https://developer.apple.com/videos/play/wwdc2026/388/)

_Platforms:_ macOS, iOS, iPadOS

## Overview
This session introduces a new end-to-end workflow for tracking down hard-to-reproduce game performance issues. The core insight is that games need rich, long-running performance data collected in real conditions—not just short Instruments captures at a desk—and that data is only actionable when correlated with what the game was actually doing at each moment.

Apple has added always-on system-level Metal performance data collection on both macOS and iOS, accumulating days of frame rate, GPU time, and related metrics without requiring the developer to initiate a capture. This data is retrievable via a new command-line tool (`metalperftrace`) or by opening the trace in Instruments, enabling investigation of issues reported by players hours after they occurred.

The new `StateReporting` framework ties game-specific context—level name, graphics preset, enemy count, any metadata—directly into the performance timeline. This transforms raw FPS drops into immediately actionable information: "frame rate fell to 42 FPS when graphics preset was High and the player entered the volcano biome."

## Key Topics

### Metal Performance Metrics (1:51)
The Metal Performance HUD surfaces the key metrics available for game analysis: FPS, on-GPU time, frame interval, CPU begin-to-present latency, next-drawable wait time, layer resolution, composition mode, and MetalFX-specific metrics. Understanding which metric is elevated is the first step in diagnosis.

### Trace Collection (3:32)
Two complementary collection methods are now available:
- **Instruments** with the Game Performance Overview template — for desk-side testing with full interactive visualization
- **Always-on system collection** (new on macOS and iOS) — the system continuously records Metal performance metrics in a rolling window, accessible after the fact without a developer-initiated capture; on iOS this is triggered via a new Control Center action

### Analyzing Performance Traces (6:38)
The `metalperftrace` CLI tool on macOS enables scripted analysis of `.atrc` trace files:
- `metalperftrace collect` — exports stored traces to disk, supporting time-range filters
- `metalperftrace overview` — prints a summary including FPS, frame time stats, GPU time, drawable wait, and shader compilation time
- JSON export mode for scripting or feeding data to AI analysis agents
- Traces can also be opened in Instruments for visual timeline inspection

### Contextualizing with StateReporting (10:08)
The new `StateReporting` framework (new in macOS/iOS 27) lets developers annotate the performance timeline with game-specific state. Developers define named domains (e.g., `com.mygame.graphics`, `com.mygame.level`) and report state transitions with optional stable and volatile metadata. This data appears as tracks in Instruments, in the Metal Performance HUD, and in `metalperftrace overview` output. The `--aggregate` flag to `metalperftrace overview` computes per-state performance statistics, pinpointing which combination of settings and content causes frame rate problems.

### Field Data with MetricKit (17:48)
New in macOS and iOS 27, MetricKit surfaces Metal frame rate data and other performance metrics from the field. StateReporting domains flow through MetricKit, so per-state performance breakdowns are available from real player devices without requiring a connected trace capture.

## APIs & Frameworks

### StateReporting (NEW framework)
- **[NEW]** `SRStateReporter` — reports game state transitions and metadata to the performance subsystem
- **[NEW]** `+[SRStateReporter reporterForDomain:]` — creates a reporter for a named domain string
- **[NEW]** `-reportTransitionToStateLabel:stableMetadata:volatileMetadata:` — marks a state transition with label and optional dictionaries
- **[NEW]** `-reportVolatileMetadataUpdate:` — updates volatile metadata without a full state transition
- Domain strings — reverse-DNS namespaced state domain identifiers (e.g., `com.mygame.level`)
- State labels — human-readable strings identifying discrete game states (e.g., `"Level 1"`)
- Stable metadata — `NSDictionary` key/value data constant for the duration of the state
- Volatile metadata — `NSDictionary` key/value data that changes frequently within a state

### MetricKit (enhanced)
- **[NEW]** Metal frame rate metrics via `MXMetricPayload` — available macOS / iOS 27
- Per-StateReporting-domain performance breakdowns in MetricKit payloads

### Instruments / Metal Debugger
- Game Performance Overview template — Instruments template for interactive game profiling
- Metal Performance HUD — live on-device overlay for FPS, GPU time, and MetalFX metrics
- StateReporting tracks in Instruments timeline — correlates game state to performance data

### metalperftrace CLI (NEW, macOS)
- **[NEW]** `metalperftrace collect <path> --last <duration>` — export stored trace for last N hours
- **[NEW]** `metalperftrace collect <path> --start <datetime> --end <datetime>` — export explicit range
- **[NEW]** `metalperftrace overview <path>` — print FPS, frame time, GPU time, memory stats
- **[NEW]** `metalperftrace overview <path> --include-state-transitions` — include StateReporting data
- **[NEW]** `metalperftrace overview <path> --aggregate [--domain <d>] [--state-label <s>]` — per-state aggregate metrics
- JSON output flag — machine-readable output for scripting and AI agents

## Code Highlights

Reporting state transitions with StateReporting:

```objc
NSString *domain = @"com.mygame.level";
SRStateReporter *reporter = [SRStateReporter reporterForDomain:domain];

[reporter reportTransitionToStateLabel:@"Level 1"
                        stableMetadata:@{ @"id": @1001 }
                      volatileMetadata:nil];

[reporter reportVolatileMetadataUpdate:@{ @"health": @100 }];
```

Collecting and analyzing a trace via CLI:

```bash
# Collect the last 5 hours of stored data
metalperftrace collect /tmp --last 5h

# Print overview with state breakdown
metalperftrace overview /Data/MyGameTrace.atrc --include-state-transitions

# Aggregate FPS by graphics preset
metalperftrace overview /Data/MyGameTrace.atrc --aggregate \
  --domain com.mygame.graphics --state-label "High"
```

## Takeaways
- Always-on system-level trace collection means performance issues players report can be investigated after the fact without a live capture session.
- `StateReporting` is the key new API: without it, traces show raw FPS numbers; with it, they show which level, preset, and game state caused the problem.
- The `metalperftrace` CLI supports scripting and AI-agent workflows, making it easy to automate regression detection across builds.
- MetricKit in macOS/iOS 27 extends field performance monitoring to include Metal frame rate and per-state breakdowns from production players.

---
_Source: WWDC26 Session 388 page (abstract, chapter summaries, code samples, and resource links)._
