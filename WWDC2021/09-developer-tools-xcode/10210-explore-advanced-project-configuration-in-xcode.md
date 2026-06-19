# Explore Advanced Project Configuration in Xcode
**WWDC21 · Session 10210** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10210/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session is a comprehensive guide to advanced Xcode project configuration across three areas: multiplatform framework targets (new in Xcode 13), best practices for scheme configuration and dependency management with custom build phases and rules, and a deep dive into build settings including xcconfig file syntax.

The multiplatform framework feature allows a single framework target to build for multiple platforms, eliminating the maintenance burden of keeping separate per-platform frameworks in sync. The build phase and rule section covers how to improve build performance via parallel builds, extract per-file transformations into build rules, use xcfilelist files for large file lists, and avoid duplicate work via aggregate targets. The build settings deep dive explains the evaluation stack (from default SDK through project-level xcconfig to target-level settings), xcconfig syntax for conditional settings, build setting composition, string/path operators, and how to use optional includes for environment-specific overrides (e.g., CI vs. local development).

## Key Topics
- **Multiplatform Framework Targets (NEW):** Set `Supported Platforms` to `Any Platform` and `Allow Multiplatform Builds = YES` on a framework target to build it once per required platform. Use `Platform Filters` on individual source files in the Compile Sources build phase to restrict them to specific platforms (e.g., macOS-only files). Replaces maintaining N separate per-platform framework targets.
- **Scheme Build Options:** Use `Dependency Order` (not deprecated `Manual Order`) to enable parallel builds based on the dependency graph. Enable `Find Implicit Dependencies` to let Xcode auto-derive cross-project dependencies from linker flags and library names.
- **Build Rules:** Extract per-file transformations (one input → one output) from script phases into build rules to enable parallel processing. Set a file pattern (e.g., `*.recipe`), declare output file paths under `DERIVED_FILE_DIR`, and optionally uncheck `Run once per architecture` for architecture-independent outputs. Add the input files to the Compile Sources build phase for the rule to be invoked.
- **Script Phase Best Practices:** Always declare input and output file dependencies; the build system cannot safely parallelize script phases without them. Use `xcfilelist` (`.xcfilelist`) files for large input lists rather than enumerating files inline. Write generated outputs to `DERIVED_FILE_DIR`, not `SRCROOT`.
- **Aggregate Targets for Deduplication:** When a multiplatform target would run the same script phase multiple times (once per platform variant), move the script to an Aggregate target. The shared framework target declares a dependency on it; the work runs exactly once.
- **Build Settings Levels:** Evaluated left-to-right from resolved value → target xcconfig → target project settings → project xcconfig → project settings → SDK defaults. Bold values indicate an explicit override at that level.
- **xcconfig Syntax:** Assignment (`SETTING = value`), conditional assignment with `[config=]`, `[arch=]`, `[sdk=]` (wildcards allowed), build setting evaluation (`$(OTHER_SETTING)`), `$(inherited)` to append to existing values, dynamic setting name composition (`$(MY_SETTING_$(CONTROL))`), string operators (`:quote`, `:lower`, `:upper`, `:identifier`), path operators (`:dir`, `:file`, `:base`, `:suffix`, `:standardizepath`), replacement operators (`:dir=/path`, etc.), `:default=` for fallback, required `#include`, optional `#include?`.

## APIs & Frameworks

**Xcode 13 Build System**
- Multiplatform Framework target type **[NEW]** – `Allow Multiplatform Builds` build setting
- Platform Filters on Compile Sources build phase entries **[NEW]** – Per-file platform restrictions
- Build Rules tab – Per-file-pattern transformation rules with declared inputs/outputs
- `xcfilelist` (`.xcfilelist`) file type – External file listing build phase inputs/outputs
- Aggregate Target type – For deduplicating work shared across multiple platform variants

**Key Build Settings**
- `SUPPORTED_PLATFORMS` – Now accepts `Any Platform` for multiplatform targets **[NEW]**
- `ALLOW_MULTIPLATFORM_BUILDS` – Set automatically when `Any Platform` is chosen **[NEW]**
- `DERIVED_FILE_DIR` – Platform-specific temp directory; recommended for generated file output
- `PROJECT_TEMP_DIR` – Project-level temp directory
- `SRCROOT` – Source root; avoid writing generated files here

**Script Phase Environment Variables**
- `SCRIPT_INPUT_FILE_COUNT` / `SCRIPT_INPUT_FILE_n` – Input files from Input Files table
- `SCRIPT_INPUT_FILE_LIST_COUNT` / `SCRIPT_INPUT_FILE_LIST_n` – Resolved input xcfilelist paths
- `SCRIPT_OUTPUT_FILE_COUNT` / `SCRIPT_OUTPUT_FILE_n` – Output file paths
- `SCRIPT_OUTPUT_FILE_LIST_COUNT` / `SCRIPT_OUTPUT_FILE_LIST_n`

**Build Rule Environment Variables**
- `SCRIPT_INPUT_FILE` – Absolute path of the current input file being processed
- `OTHER_INPUT_FILE_FLAGS` – Custom flags from Compile Sources for the input file
- `SCRIPT_INPUT_FILE_n` / `SCRIPT_OUTPUT_FILE_n` – Additional inputs/outputs
- `SCRIPT_HEADER_VISIBILITY` / `HEADER_OUTPUT_DIR` – For header file processing

## Code Highlights
Build rule output file path (using `DERIVED_FILE_DIR`):
```
$(DERIVED_FILE_DIR)/$(INPUT_FILE_BASE).compiledrecipe
```

Build rule script:
```bash
"$SRCROOT/Scripts/gen-code.sh" "$SCRIPT_INPUT_FILE" "$SCRIPT_OUTPUT_FILE_0"
```

xcconfig: conditional build setting with SDK wildcard:
```
MY_SETTING = default_value
MY_SETTING[config=Debug] = -debug_flag
MY_SETTING[sdk=iphone*] = -ios_only
```

xcconfig: dynamic build setting composition:
```
IS_ENABLED = NO
MY_SETTING_NO  = -use_this_one
MY_SETTING_YES = -use_this_instead
MY_SETTING = $(MY_SETTING_$(IS_ENABLED))
```

xcconfig: optional include for CI override:
```
// common.xcconfig
#include? "ci.xcconfig"
OTHER_SWIFT_FLAGS = $(inherited) -Xfrontend -warn-long-expression-type-checking=$(MAX_EXPRESSION_TIME:default=200)

// ci.xcconfig  
MAX_EXPRESSION_TIME = 500
```

## Takeaways
- Multiplatform framework targets eliminate the N-target maintenance problem; platform filters let you still gate individual source files to specific platforms without splitting the target.
- Always use `Dependency Order` in scheme build settings and declare all input/output dependencies on script phases—without them, Xcode must serialize builds conservatively, negating parallel build gains.
- Build rules are the right abstraction for per-file transformations (many inputs, one output per input); script phases handle the case where all inputs must be processed together.
- xcconfig files beat inline project settings for complex build configurations: they support conditional syntax, dynamic composition, optional includes, and source control diff-ability.

---
_Source: WWDC21 Session 10210 page (abstract, chapter summaries, code samples, and resource links)._
