# Build AI-Powered Scripts with the fm CLI and Python SDK
**WWDC26 · Session 334** · [Watch](https://developer.apple.com/videos/play/wwdc2026/334/)

_Platforms:_ macOS 27+

## Overview
Session 334 introduces two entirely new ways to access Apple Foundation Models on macOS that complement the Swift framework: the `fm` command-line tool (pre-installed with macOS 27) and the `apple_fm_sdk` Python package. Together they make the on-device model and Private Cloud Compute accessible to shell scripts, Jupyter notebooks, data science pipelines, and any workflow where reaching for Xcode is not the right tool.

The `fm` CLI brings the power of Apple Intelligence to the terminal for everyday developer productivity: interactive sessions via `fm chat`, one-shot inline prompting via `fm respond`, structured JSON output via `fm schema`, and PCC access with `--model pcc`. The Python SDK mirrors the Swift framework's API — `LanguageModelSession`, tool calling, guided generation with `@fm.generable` — and is particularly useful for ML engineers who want to run evaluation pipelines in Jupyter alongside pandas and matplotlib.

Both tools are designed for quick prototyping and automation rather than shipping App Store products, though the evaluation pipeline use case is central: the session demo uses the Python SDK with a judge model to score three prompt variants for a grocery-cart feature.

## Key Topics

### Introducing the fm CLI and Python SDK (1:22)
`fm` ships as part of macOS 27 with no installation required. The Python SDK requires Python 3.10+, Xcode, and Apple Silicon, and is installed from PyPI. Both access the same on-device model as the Swift framework; both can target PCC.

### Command Line Tool: fm chat (3:23)
`fm chat` opens an interactive REPL-style conversation with the on-device model. Switch to PCC mid-session with `--model pcc`. Sessions can be saved and resumed, making it practical for iterative prompt refinement. Pipes standard Unix stdin/stdout for scripting.

### fm respond and Structured Output (5:02)
`fm respond "<prompt>"` returns the model's answer as plain text to stdout. Options: `--model pcc`, `--image <file>` for multimodal input, `--instructions <text>` for system prompts, `--schema <json-file>` for structured JSON output. Generate a schema with `fm schema object --name <Name> --string <field> --array`.

### Automating File Management with fm (6:11)
Practical shell-script demo: generate a schema for `TriagedFileList`, call `fm respond` with a file listing as input, parse the JSON output with `jq`, and route files to `final_files` and `draft_files` directories. The model understands naming conventions (version numbers, "draft", "final" keywords) to classify files without hard-coded rules.

### Python SDK Core API (8:52–9:42)
`pip install apple_fm_sdk`. Create a session with `fm.LanguageModelSession(instructions=..., tools=...)`, call `await session.respond(prompt)` for plain text. Define tools by subclassing `fm.Tool` with a `name`, `description`, `@fm.generable`-decorated `Arguments` class, and an async `call(args:)` method. Generate structured output by passing a `@fm.generable` class to `session.respond(prompt, generating=MyType)`.

### Building an Evaluation Pipeline in Python (10:44)
End-to-end demo with Jupyter + pandas + matplotlib: define three prompt variants for a grocery cart feature, generate outputs using the on-device `SystemLanguageModel`, score them with a server judge model (PCC) on three criteria (excess items, missing items, hallucinations), load results into a DataFrame, and visualise with bar charts to pick the best prompt. This mirrors the Evaluations framework workflow but in the Python ecosystem.

## APIs & Frameworks

**fm CLI (macOS 27, pre-installed)**
- `fm chat` **[NEW]** — interactive conversation
- `fm respond "<prompt>"` **[NEW]** — one-shot response
  - `--model pcc` — use Private Cloud Compute
  - `--image <file>` — multimodal input
  - `--instructions <text>` — system instructions
  - `--schema <json-file>` — structured JSON output
  - `--help`
- `fm schema object` **[NEW]** — generate a JSON schema
  - `--name <Name>`
  - `--string <field> --array` — string array field
- Session save/resume **[NEW]**

**apple_fm_sdk (Python, `pip install apple_fm_sdk`)**
- `fm.SystemLanguageModel()` **[NEW]**
- `fm.LanguageModelSession(model=, instructions=, tools=)` **[NEW]**
- `await session.respond(prompt)` → `str` **[NEW]**
- `await session.respond(prompt, generating=MyType)` → structured output **[NEW]**
- `model.is_available()` → `(bool, reason)` **[NEW]**
- `@fm.generable("description")` decorator **[NEW]** — marks a class as structured output
- `fm.guide("description")` **[NEW]** — field-level generation guidance
- `fm.Tool` base class **[NEW]**
  - `name: str`, `description: str`
  - `Arguments` inner class with `@fm.generable`
  - `arguments_schema` property → `fm.GenerationSchema`
  - `async call(self, args: fm.GeneratedContent) -> str`
- `fm.GeneratedContent` **[NEW]** — `.value(type, for_property=)` accessor
- `fm.GenerationSchema` **[NEW]**

**Ecosystem integrations (demonstrated)**
- Jupyter notebooks
- pandas (DataFrame for results)
- matplotlib (bar charts)

## Code Highlights

```bash
# Structured output from the shell
fm schema object --name "TriagedFileList" \
    --string 'final_files' --array \
    --string 'draft_files' --array > /tmp/schema.json

output=$(fm respond \
    --instructions "Triage these files into final vs draft versions." \
    "Files: ${files_list}" \
    --schema /tmp/schema.json)

echo "${output}" | jq -r '.final_files[]' | while read -r file; do
    cp "${DIR}/${file}" "${FINAL_DIR}"
done
```

```python
# Python SDK — structured generation with tool calling
@fm.generable("Suggested items")
class ItemsSuggestion:
    item_names: list[str] = fm.guide("Names of the suggested items")

session = fm.LanguageModelSession(instructions=INSTRUCTIONS, tools=load_tools())
result = await session.respond(prompt, generating=ItemsSuggestion)
```

## Takeaways
- Use `fm respond` in shell scripts to add model-powered classification, summarisation, and extraction without writing any Swift — combine with `jq` for structured JSON workflows.
- The Python SDK enables ML engineers to prototype Foundation Models features, run evaluation pipelines, and visualise results entirely in Jupyter without touching Xcode.
- Use the Python SDK's judge-model pattern (on-device model generates, PCC scores) as a fast, cheap evaluation loop before investing in a full Swift `Evaluation` suite.
- Both tools access the same on-device model as the Swift framework, so insights from scripting translate directly to the shipping app.

---
_Source: WWDC26 Session 334 page (abstract, chapter summaries, code samples, and resource links)._
