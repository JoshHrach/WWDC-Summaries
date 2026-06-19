# Debug and Profile Agentic App Experiences with Instruments

**WWDC26 · Session 243** · [Watch](https://developer.apple.com/videos/play/wwdc2026/243/)

_Platforms:_ Xcode 27 · iOS · macOS · iPadOS · Latest OS required

## Overview

This session introduces the enhanced Foundation Models Instruments template in Xcode 27, designed specifically for debugging and profiling apps built with the Foundation Models framework. Standard debugging tools are insufficient for LLM-powered apps because their behavior is probabilistic (non-deterministic responses break unit tests), they often involve multiple coordinated models, and the control flow across inference calls is difficult to observe. The Foundation Models instrument addresses all three challenges.

The session walks through a journaling app demo ("Craft") that uses two sets of Dynamic Instructions — one for idea generation, one for tutorial creation — both backed by the server model on Private Cloud Compute. Using the Foundation Models template in Instruments, the developer records a trace, then navigates tracks and lanes on the timeline (including the instructions lane and model inference lane with yellow/orange bars) to inspect individual sessions, requests, inferences, and tool calls in a tree view.

The performance metrics chapter defines three key metrics for LLM experience optimization: time-to-first-token (reduce by shortening prompts), tokens-per-second (benchmark across configurations), and total latency (reduce perceived wait with streaming). These metrics provide a concrete framework for making agentic experiences feel fast and responsive.

## Key Topics

### LLM App Development Mindset
Three unique challenges compared to traditional app development:
1. **Probabilistic output** — non-deterministic responses make standard unit testing unreliable.
2. **Model-to-model communication** — data flow coordination across multiple model sessions is complex.
3. **Observability** — understanding where things went wrong in a multi-model pipeline requires specialized tooling.

### Recording a Trace with Instruments
- Select the Foundation Models template in Instruments.
- Record a session while exercising the agentic feature in the app.
- Important: trace files may contain sensitive prompt data — handle accordingly.

### Navigating the Instruments UI
- **Timeline tracks and lanes**: Instructions lane and model inference lane (yellow/orange bars indicate active inference).
- **Detail view**: tree structure of sessions → requests → inferences → tool calls.
- **Inspector panel**: full content of prompts, responses, tool call inputs/outputs, and timing metadata.
- Navigate from the timeline to the detail view to the inspector for root cause investigation.

### Performance Metrics
Three key metrics for LLM experience optimization:
- **Time-to-first-token**: first visible output; reduce by shortening system prompts and context.
- **Tokens-per-second**: generation throughput; benchmark across model configurations.
- **Total latency**: end-to-end response time; reduce perceived wait with streaming (`AsyncStream`).

## APIs & Frameworks

**Foundation Models Framework**
- `LanguageModelSession` — primary session type; multiple instances shown in the timeline tree
- **[NEW]** `DynamicInstructions` — configurable instruction sets applied per-session; shown in the instructions lane
- Tool calls — agent-invoked functions visible as nodes in the detail tree
- Private Cloud Compute (server model) — remote inference backend; latency appears in inference lane
- `AsyncStream` — streaming token delivery to reduce perceived total latency

**Instruments (Xcode 27)**
- **[NEW/ENHANCED]** Foundation Models instrument template
- Instructions lane — timeline lane showing Dynamic Instructions activity
- Model inference lane — timeline lane with yellow/orange bars per inference call
- Session tree: Sessions → Requests → Inferences → Tool Calls
- Inspector panel — full prompt and response content with timing metadata
- Time-to-first-token measurement
- Tokens-per-second measurement

**Related Documentation**
- [Analyzing the runtime performance of your Foundation Models app](https://developer.apple.com/documentation/FoundationModels/analyzing-the-runtime-performance-of-your-foundation-models-app)

**Related Sessions (WWDC26)**
- Build agentic app experiences with the Foundation Models framework (242)
- What's new in the Foundation Models framework (241)
- Build with the new Apple Foundation Model on Private Cloud Compute (319)

## Code Highlights

No explicit code samples in this session. Key instrumentation patterns:

- Use multiple `LanguageModelSession` instances for distinct agent roles; Instruments displays each as a separate track, making parallel vs. sequential execution clearly visible.
- Enable streaming on all inference calls to improve time-to-first-token perception even when total latency is unchanged.
- Keep system prompts and Dynamic Instructions concise — they directly affect time-to-first-token.

## Takeaways

- The Foundation Models instrument template is the essential debugging tool for any app using the Foundation Models framework; standard breakpoints and logs are insufficient for non-deterministic LLM behavior.
- Trace files contain full prompt and response content — treat them as sensitive data, especially in production debugging.
- The three-metric framework (time-to-first-token, tokens-per-second, total latency) gives concrete, measurable targets for LLM UX optimization.
- Streaming token delivery is the most impactful single change for improving perceived responsiveness without reducing actual latency.

---
_Source: WWDC26 Session 243 page (abstract, chapter summaries, and resource links)._
