# Creating Custom Instruments
**WWDC18 · Session 410** · [Watch](https://developer.apple.com/videos/play/wwdc2018/410/)

_Platforms:_ iOS, macOS, watchOS, tvOS (Instruments 10 / Xcode 10)

## Overview
Instruments 10 introduces a fully standardized architecture that allows third-party developers to create custom instruments with the same capabilities as the instruments that ship with Xcode. This session explains the architecture (Standard UI + Analysis Core), walks through creating an Instruments Package project in Xcode, and shows three levels of complexity: a basic tick-counting instrument, an `os_signpost`-driven networking instrument with graphs and aggregations, and an advanced CLIPS-language modeler that detects anti-patterns as an expert system.

The key insight is that every built-in Instruments instrument — Time Profiler, System Trace, Network Connections — is built entirely from the Standard UI and Analysis Core. Custom instruments use exactly the same building blocks.

## Key Topics

### Architecture: Standard UI and Analysis Core
- **Analysis Core** — a combined time-series database and expert system; all instrument data lives here as tables.
- **Standard UI** — renders track view (graphs/lanes) and detail view; tightly coupled with the Analysis Core; enforces consistent visual treatments across all instruments.
- Custom instruments = custom XML configuration of both Standard UI and Analysis Core.
- **Tables** — collections of rows with a schema (defines column names and engineering types) and optional key-value attributes.
- **Engineering types** — rich type system that controls both storage and visualization (e.g., `sample-time`, state, magnitude, backtrace, narrative).
- **Schemas** — reusable table definitions; Instruments ships with 100+ schemas in system packages; importable via `<import-schema>`.
- **Binding solution** — computed by Instruments at record time; maps tables to data providers (direct recording or modelers); shared trace-wide to minimize target impact.

### Instruments Package Project
- New Xcode project type: "Instruments Package" (macOS target).
- Package definition is XML (`.instrpkg` file); contains `<package>`, `<instrument>`, `<schema>`, `<modeler>`, `<template>` elements.
- Build and run the package target: launches a special Instruments instance that loads the package temporarily for iteration.
- Package Management UI in Instruments Preferences shows installed packages and debug batches.
- Linked Instruments Packages build setting — required when referencing schemas from non-base packages.

### Graph Lanes (Track View)
- `<graph>` element with one or more `<lane>` children.
- `<plot>` — static; graphs all rows in the referenced table against the targeted column.
  - Schema type determines visual treatment: interval+magnitude → bar graph; interval+state → rounded-rectangle state lanes.
- `<plot-template>` — dynamic; creates one lane per unique value of a specified column (`<instance-by>`).
- `<histogram>` — buckets the timeline into fixed intervals; uses aggregate functions (count, sum, min, max) to show spikes.

### Detail Views
- `<list>` — flat table view of all rows; specify columns to show.
- `<aggregation>` — statistical summary; columns are functions (sum, count, average); supports multi-level `<hierarchy>`; supports `<slice>` to filter rows by column predicate.
- `<call-tree>` — weighted backtrace tree (requires backtrace column and weight column); equivalent to Time Profiler's call tree.
- `<narrative>` — displays narrative-typed column values as diagnostic text; interactive (filterable, clickable).
- `<time-slice>` — list filtered to rows intersecting the inspection head.

### os_signpost Integration (os Signpost Interval Schema)
- `<os-signpost-interval-schema>` element — defines a schema AND auto-generates a modeler from `os_signpost` begin/end calls.
- Specify `<subsystem>`, `<category>`, `<name>` to match signpost log handle and call name.
- `<start-pattern>` / `<end-pattern>` — capture metadata variables from signpost message format strings.
- `<column>` with `<expression>` — CLIPS expression that computes the column value from captured variables.

### Advanced Modeling with CLIPS
- Modelers are miniature expert systems written in the CLIPS rule language.
- **Working memory** — set of currently active facts; time-bounded by the modeler's clock (current input's timestamp).
- **Modeler clock** — advances to each input's timestamp; facts whose intervals no longer intersect the clock are retracted.
- **Production rules** — `(defrule name (LHS pattern...) => (RHS action...))` — fire when LHS pattern matches working memory.
- **Fact templates** — `(deftemplate name ...)` — structured facts with named slots and types.
- **Modules** — `modeler` module rules run before `recorder` module rules; controls execution order and output timing.
- **Logical support** — `(logical ...)` wrapper on LHS pattern: if supporting facts are retracted, the asserted fact is automatically retracted (avoids working memory bloat).
- Two rule examples: `found-cause` (assert inferred fact) and `record-cause` (write row to output table).
- Inputs to a modeler: any existing table (recorded directly or produced by another modeler); outputs: point or interval schema tables.

### Recording Modes and Best Practices
- **Immediate mode** — real-time visualization as data arrives; difficult with interval-schema inputs (open intervals block modeler clock).
- **Deferred mode** — all data processed after recording stops.
- **Last N seconds mode** — most efficient for high-volume data (up to 10× faster for signpost data); use for System Trace-style instruments.
- Opt out of immediate mode via `<limitation>` element in the instrument definition.
- Write multiple fine-grained instruments rather than one large instrument; let users compose via templates.
- Save a configured trace document as a custom template via File > Save as Template; reference in package via `<template>` element.

## APIs & Frameworks

**Instruments Package XML Elements (new, Xcode 10)**
- `<package>` — `identifier`, `title`, `owner`
- `<import-schema>` — `name` (reference system schema by name)
- `<instrument>` — `id`, `title`, `category`, `purpose`, `icon`
- `<create-table>` — `id`, `schema-ref`, optional `attribute` key-value pairs
- `<graph>` — `title`; contains `<lane>` elements
- `<lane>` — `title`, `table-ref`; contains `<plot>`, `<plot-template>`, or `<histogram>`
- `<plot>` — `value` (column mnemonic)
- `<plot-template>` — `instance-by` (column), `label-format`, `value`, `color`, `label`
- `<histogram>` — `slice`, `value`, `color`
- `<list>` — `title`, `table-ref`, `column` children
- `<aggregation>` — `title`, `table-ref`, `slice`, `hierarchy`, `column` (with function attribute)
- `<call-tree>` — `title`, `table-ref`, `weight`, `backtrace`
- `<narrative>` — `title`, `table-ref`, `time`, `message`
- `<time-slice>` — `title`, `table-ref`, `column` children
- `<os-signpost-interval-schema>` — `id`, `title`, `subsystem`, `category`, `name`, `start-pattern`, `end-pattern`, `column` (with `expression`)
- `<modeler>` — `id`, `title`, `purpose`, `production-system` (path to `.clp` file), `output-schema`, `required-input`
- `<point-schema>` / `<interval-schema>` — output schema types for modelers
- `<template>` — embed a custom Instruments template in a package
- `<limitation>` — opt instrument out of immediate mode

**CLIPS Language (used in modeler production systems)**
- `(deftemplate ...)` — define a fact template
- `(defrule ...)` — define a production rule
- `(assert ...)` — add a fact to working memory
- `(retract ...)` — remove a fact from working memory
- `(logical ...)` — logical support wrapper
- Modules: `modeler`, `recorder`

**os Logging (data source)**
- `os_log_create(subsystem:category:)` — create log handle
- `os_signpost_id_make_with_pointer(_:_:)` / `OSSignpostID` — create unique signpost ID
- `os_signpost(.begin, ...)` / `os_signpost(.end, ...)` — mark interval boundaries with metadata

## Code Highlights

Minimal Instruments Package definition importing the `tick` schema:
```xml
<package identifier="com.example.ticks" title="Ticks" owner="Example">
    <import-schema>tick</import-schema>
    <instrument id="com.example.ticks.instrument" title="Ticks" category="Behavior"
                purpose="Graphs clock ticks at 10ms intervals" icon="Hitch">
        <create-table id="tick-table" schema-ref="tick">
            <attribute key="frequency" value="100"/>
        </create-table>
        <graph title="Ticks">
            <lane title="Ticks" table-ref="tick-table">
                <plot value="time"/>
            </lane>
        </graph>
        <list title="All Ticks" table-ref="tick-table">
            <column>time</column>
        </list>
    </instrument>
</package>
```

os_signpost interval schema capturing metadata from signpost messages:
```xml
<os-signpost-interval-schema id="image-download" title="Image Download">
    <subsystem>com.apple.trailblazer</subsystem>
    <category>networking</category>
    <name>Background Image</name>
    <start-pattern>
        <message>img-name %{public,name}s caller %{caller}p</message>
    </start-pattern>
    <end-pattern>
        <message>status %{status}s size %{size}d</message>
    </end-pattern>
    <column id="image-name" title="Image" type="string">
        <expression>name</expression>
    </column>
    <column id="status" title="Status" type="string">
        <expression>status</expression>
    </column>
</os-signpost-interval-schema>
```

CLIPS rule detecting overlapping network requests (anti-pattern detection):
```clips
(deftemplate started-download
    (slot time) (slot caller) (slot signpost-id) (slot image-name))

(defrule modeler::track-download
    (os-signpost (subsystem "com.apple.trailblazer") (name "Background Image")
                 (event-type begin) (image-name ?img) (caller ?addr)
                 (time ?t) (id ?sid))
    =>
    (assert (started-download (time ?t) (caller ?addr)
                              (signpost-id ?sid) (image-name ?img))))

(defrule recorder::detect-overlap
    (started-download (time ?t1) (caller ?addr) (image-name ?img1))
    (started-download (time ?t2) (caller ?addr) (image-name ?img2))
    (test (< ?t2 ?t1))
    ?out <- (output-table (schema "downloader-narrative"))
    =>
    (create-row ?out (time ?t1)
                    (description (str-cat "Cell reused while download running: " ?img1))))
```

## Takeaways
- Custom Instruments in Xcode 10/Instruments 10 use the same Standard UI + Analysis Core as all built-in instruments; what differs is only who wrote the package.
- `<os-signpost-interval-schema>` is the lowest-friction path to a custom instrument: instrument your code with `os_signpost`, declare the schema in XML, and a modeler is auto-generated.
- CLIPS-based modelers are full expert systems — they can detect temporal patterns, correlate events across multiple tables, and output narrative diagnostics without you being present during profiling.
- Prefer deferred or "last N seconds" recording for high-volume instruments; immediate mode with interval-schema inputs causes modeler clock stalls visible to users.

---
_Source: WWDC18 Session 410 page (abstract, full transcript, and resource links)._
