# Xcode, Agents, and You

**WWDC26 · Session 259** · [Watch](https://developer.apple.com/videos/play/wwdc2026/259/)

_Platforms:_ Xcode 27 · iOS · macOS · iPadOS

## Overview

This session presents a full development lifecycle walkthrough using Xcode 27's coding agents, organized around four practical phases: Explore, Build, Refine, and Orchestrate. Using a workout tracking app as the running example, the session demonstrates how agents function as active collaborators rather than autonomous code generators — keeping the developer in creative control at every stage.

The Explore phase shows how to rapidly get up to speed on an unfamiliar codebase: agents can produce walkthrough explanations of data models and view hierarchies, answer framework questions via Apple Document Search, and help capture architectural knowledge as reusable documentation files. The Build phase introduces plan mode, where the agent designs an architecture before writing any code, and queued messages, which let developers communicate changes in real time while the agent works.

The Refine and Orchestrate phases demonstrate more advanced usage: attaching images to convey design intent, using inline source annotations to direct targeted edits, and describing high-level goals like localization or accessibility so Xcode can coordinate parallel sub-agents to accomplish them concurrently.

## Key Topics

### Explore
- Agent-driven codebase walkthroughs: data models, view hierarchies.
- Apple Document Search for accurate framework-specific knowledge.
- Capturing learnings as reusable architecture documents in the project.

### Build
- **Plan mode**: agent proposes an architecture and file structure before writing code.
- **Queued messages**: queue follow-up instructions while the agent is actively working.
- Integration with Xcode build, Preview, and test tools to validate features as they are implemented.

### Refine
- Iterating on visual design with Swift Charts using realistic preview data.
- **Image attachments**: paste or drag screenshots into the agent chat to convey design intent without writing lengthy descriptions.
- **Inline annotations**: add comments directly in source code to direct targeted, context-aware changes.

### Orchestrate
- Describe a high-level goal (e.g., "localize the app", "improve accessibility").
- Xcode discovers the correct tools and coordinates **sub-agents** running in parallel.
- Results arrive faster than sequential single-agent execution.

## APIs & Frameworks

**Xcode 27 Coding Agent Features**
- **[NEW]** Plan mode (`/plan` command) — architecture-first agent workflow
- **[NEW]** Queued messages — send follow-up instructions while agent is running
- **[NEW]** Image attachments in agent chat — paste screenshots/mockups to convey design intent
- **[NEW]** Inline source annotations — comment-directed targeted changes
- **[NEW]** Sub-agent orchestration — parallel multi-agent task execution
- **[NEW]** Apple Document Search — agent queries framework documentation for accurate API knowledge
- **[NEW]** Architecture document capture — agents can write/update project docs
- Agent conversations as first-class editor tabs with split-view support

**Validated via**
- Xcode Previews (real-time visual feedback)
- Xcode build system (compile-time correctness)
- Swift Testing / XCTest (automated test validation)

**Swift Charts**
- Used for iterative visual design with realistic data in previews

**Related Documentation**
- [Writing code with intelligence in Xcode](https://developer.apple.com/documentation/Xcode/writing-code-with-intelligence-in-xcode)

## Code Highlights

No explicit code samples in this session. The session is a workflow demonstration. Key patterns:

- Use `/plan` before asking the agent to write code to get a proposed architecture first.
- Attach a screenshot directly to the agent chat instead of describing UI changes in text.
- Place a comment in source (e.g., `// Agent: make this transition smoother`) to scope edits to a specific location.
- For large tasks (localization, accessibility audit), express the goal at a high level and let Xcode orchestrate sub-agents rather than breaking it down manually.

## Takeaways

- The four-phase model (Explore → Build → Refine → Orchestrate) maps cleanly to the real development lifecycle and is applicable to both solo and team workflows.
- Plan mode prevents the most common agent failure mode — generating the wrong architecture — by separating design from implementation.
- Image attachments and inline annotations are the highest-leverage techniques for keeping visual and contextual intent precise without lengthy prose prompts.
- Sub-agent orchestration for high-level tasks (localization, accessibility) removes the manual decomposition step, making previously time-consuming housekeeping practical to delegate.

---
_Source: WWDC26 Session 259 page (abstract, chapter summaries, and resource links)._
