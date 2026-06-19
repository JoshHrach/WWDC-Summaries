# Explore prompt design and safety for on-device foundation models

**Session ID:** 248  
**WWDC Year:** 2025  
**Folder:** `04-ai-foundation-models`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/248/

---

## Overview

This session covers best practices for using Apple's on-device Foundation Models framework effectively and safely. It explains how the ~3B-parameter on-device model processes instructions and context, describes the model's built-in safety system and how developers should work with it, and provides concrete guidance on prompt design: structuring system prompts, writing good instructions, using few-shot examples, and chaining sessions for complex tasks. The session also covers the Feedback API for collecting user corrections and the `Guardrails` type for explicitly controlling which safety categories to enforce.

---

## Key Topics

- How the on-device LLM interprets system prompts vs. user turns
- Prompt design principles: clarity, specificity, format instructions, few-shot examples
- The model's layered safety system and developer-facing `Guardrails` API
- Session chaining for multi-step reasoning workflows
- Collecting user feedback on model outputs with `LanguageModelFeedback`
- Diagnosing prompt issues with the `FoundationModels` Xcode instrument
- Testing prompts with the on-device Playgrounds tool

---

## APIs & Frameworks

- **Foundation Models** framework (`import FoundationModels`) – on-device ~3B LLM for Apple Intelligence; available on iOS 26, iPadOS 26, macOS 26.
- **`SystemLanguageModel`** – Entry point; call `.default` to get the shared on-device model instance.
- **`LanguageModelSession`** – Represents a conversation context; holds the transcript of prior turns for multi-turn interactions.
- **`LanguageModelSession(instructions:)`** – **[NEW]** Initializer accepting a plain-text system prompt `String` to configure model behavior for the session.
- **`session.respond(to:)`** – Async method sending a user prompt and receiving the model's `Response`.
- **`Response.content`** – The generated text string.
- **`Guardrails`** – **[NEW]** Struct for explicitly enabling or disabling individual safety categories (e.g., `.safetyCategories([.hateSpeech, .selfHarm])`).
- **`LanguageModelSession(instructions:guardrails:)`** – **[NEW]** Session initializer accepting a `Guardrails` value alongside the system prompt.
- **`LanguageModelFeedback`** – **[NEW]** Type for reporting user thumbs-up/thumbs-down on a specific response; call `session.record(_:for:)`.
- **`@Generable`** macro** – Applied to Swift structs/enums to make them structured output targets for `session.respond(to:generating:)`.
- **`FoundationModels` Xcode Instrument** – **[NEW]** Profiling instrument showing per-request latency, token counts, and safety filter decisions.
- **`LanguageModelSession.transcript`** – Array of prior turns; can be inspected or trimmed to manage context window usage.

---

## Code Highlights

Basic session with a system prompt:
```swift
import FoundationModels

let session = LanguageModelSession(
    instructions: "You are a helpful assistant that summarizes meeting notes concisely."
)
let response = try await session.respond(to: "Summarize: \(meetingNotes)")
print(response.content)
```

Configuring guardrails:
```swift
let session = LanguageModelSession(
    instructions: "You help moderate user-generated content.",
    guardrails: Guardrails(safetyCategories: [.hateSpeech, .sexualContent])
)
```

Structured output with `@Generable`:
```swift
@Generable
struct MeetingAction {
    var owner: String
    var deadline: String
    var description: String
}

let actions = try await session.respond(to: prompt, generating: [MeetingAction].self)
```

Recording user feedback:
```swift
let response = try await session.respond(to: userMessage)
// User taps thumbs-down
await session.record(LanguageModelFeedback.negative, for: response)
```

---

## Takeaways

- Clear, specific system prompts are the single highest-leverage way to improve Foundation Model output quality; vague instructions lead to inconsistent results.
- Few-shot examples in the system prompt (showing 2–3 input/output pairs) dramatically improve structured or domain-specific tasks.
- The built-in safety system cannot be fully disabled; `Guardrails` only allows narrowing or expanding which categories are enforced within what the model permits.
- Chain multiple `LanguageModelSession` instances for multi-step reasoning rather than stuffing everything into one long context.
- Use the `FoundationModels` Xcode Instrument during development to catch prompts that hit safety filters or exceed context limits before shipping.
- `LanguageModelFeedback` data stays on-device and is used for future model improvements via differential privacy; no raw text leaves the device.
