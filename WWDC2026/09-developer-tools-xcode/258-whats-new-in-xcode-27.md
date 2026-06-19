# What's New in Xcode 27

**WWDC26 · Session 258** · [Watch](https://developer.apple.com/videos/play/wwdc2026/258/)

_Platforms:_ Xcode 27 · macOS · iOS · iPadOS · tvOS · watchOS · visionOS

## Overview

Xcode 27 introduces a sweeping redesign focused on customization, agentic workflows, and post-launch quality tools. The toolbar has been rebuilt from scratch with new controls, a coding agent entry point, and full drag-to-reorder customization. A new Appearance panel lets developers dial in per-project color themes using sliders and presets, making it easy to visually distinguish workspaces at a glance.

The release tightens the integration between the coding agent and the editor: agent conversations now live in a first-class editor pane with tab and split-view support, and a new `/plan` command lets the agent scope out work before touching any code. Two new lightweight project workflows—untitled projects and standalone Swift file editing with live previews—lower the friction for rapid prototyping.

Post-launch tooling receives significant upgrades: the Organizer surfaces high-impact issues first, adds storage and animation hitch metrics, and introduces Metric Goals along with agent-powered fix recommendations. A new Top Functions view in Instruments slashes the time needed to identify expensive code paths, and Xcode Cloud gains a streamlined onboarding flow for commit-triggered builds and seamless TestFlight delivery.

## Key Topics

### Workspace & Toolbar
- Fully customizable toolbar with reorderable controls and a dedicated coding agent button.

### Themes (Appearance Panel)
- New Appearance panel with color/font sliders, preset themes, and per-project theme assignment.

### Inline Issues
- Predictive issues show in a subdued style while typing; they escalate to full warnings/errors only after a build, reducing distraction mid-keystroke.

### New Project Workflows
- Create untitled projects instantly or open standalone `.swift` files with live previews and playground-style results—no project scaffolding required.

### Coding Agents in the Editor
- Agent conversations embedded as editor tabs with full split-pane support.
- New `/plan` command to design an approach before the agent writes any code.

### Device Hub
- Unified window for running and inspecting apps on simulators and physical devices.
- Supports accessibility settings, iPhone Mirroring resize testing, and condition simulation.

### Localization
- Coding agents can set up String Catalogs, configure localization, and generate translations for multiple languages in a single agent session.

### Organizer
- Redesigned to surface high-impact issues first.
- New metrics: storage usage and animation hitches.
- **[NEW]** Metric Goals: set targets and track progress over time.
- Agent-powered fix recommendations for surfaced issues.

### Instruments & Top Functions
- **[NEW]** Top Functions view: ranks the most expensive code paths so performance regressions surface immediately without manual call-tree diving.

### Xcode Cloud
- Streamlined onboarding: connect a source repository and get builds and tests running automatically on every commit, with direct TestFlight and App Store delivery.

## APIs & Frameworks

**Xcode 27 Tools**
- **[NEW]** Appearance panel (per-project themes, color/font sliders, presets)
- **[NEW]** Customizable toolbar with reorderable items
- **[NEW]** Coding agent editor pane (tab and split-view support)
- **[NEW]** `/plan` agent command
- **[NEW]** Untitled project workflow
- **[NEW]** Standalone Swift file editing with Previews and playground results
- **[NEW]** Device Hub (unified device/simulator management window)
- **[NEW]** iPhone Mirroring resize testing in Device Hub
- **[NEW]** Metric Goals in Organizer
- **[NEW]** Storage metric in Organizer
- **[NEW]** Animation hitch metric in Organizer
- **[NEW]** Agent-powered fix recommendations in Organizer
- **[NEW]** Top Functions view in Instruments
- **[NEW]** Streamlined Xcode Cloud onboarding assistant

**Localization**
- String Catalogs (agent-assisted creation and translation)

**Xcode Cloud**
- Source repository connection
- Commit-triggered build workflows
- TestFlight distribution workflow
- App Store delivery

## Code Highlights

No code samples are shown in this session — it is a feature walkthrough. See related sessions for code-level details.

## Takeaways

- Xcode 27's per-project themes, customizable toolbar, and `/plan` agent command collectively reduce context-switching overhead throughout the day.
- The inline issue redesign (subdued predictive issues while typing, full display after build) significantly cuts noise during active editing.
- Top Functions in Instruments and expanded Organizer metrics (storage, hitches, Metric Goals) give developers faster signal on regressions both locally and in production.
- Xcode Cloud's new onboarding flow removes the multi-step setup barrier, making CI/CD adoption practical for solo developers and small teams.

---
_Source: WWDC26 Session 258 page (abstract, chapter summaries, and resource links)._
