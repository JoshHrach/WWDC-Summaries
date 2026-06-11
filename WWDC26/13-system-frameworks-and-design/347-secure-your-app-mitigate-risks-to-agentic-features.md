# Secure your app: mitigate risks to agentic features
**WWDC26 · Session 347** · [Watch](https://developer.apple.com/videos/play/wwdc2026/347/)

_Platforms:_ iOS, macOS, iPadOS, visionOS

## Overview
Agentic features — LLM-powered tools that can take actions on behalf of users — introduce new attack surfaces that traditional app security models do not address. The primary threat covered is indirect prompt injection: malicious instructions embedded in content a model reads (a web page, a document, a tool response) that cause the agent to exfiltrate data or perform unintended destructive actions.

The session walks through a structured threat-modeling exercise, then presents concrete mitigations available via two Apple frameworks: the Foundation Models framework (for on-device LLM sessions) and App Intents (for Siri/Apple Intelligence integration). Mitigations include user confirmation checkpoints, prompt design techniques (spotlighting and redaction), and authentication policies on intents.

Developers are encouraged to think of agentic systems as having two distinct trust domains: trusted instructions from the developer/user and untrusted content from the environment. The session provides concrete API patterns for enforcing that boundary.

## Key Topics

### Risks
Indirect prompt injection: untrusted content in the environment instructs the model to exfiltrate private data, post to public channels, make purchases, or delete files. Risks are amplified when tools combine high-privilege actions with access to untrusted content sources.

### Threat Modeling
Map your agent's context sources (trusted vs. untrusted) and its actions (low-risk vs. high-risk). Any flow where untrusted content can reach a high-risk tool is a vulnerability requiring a mitigation.

### Foundation Models Mitigations
- **`onToolCall` confirmation** — intercept tool calls before execution; prompt the user to approve high-risk operations (purchases, deletions)
- **`historyTransform` spotlighting** — wrap untrusted tool outputs in explicit delimiters (e.g., `<<UNTRUSTED>>…<</UNTRUSTED>>`) so the model is less likely to follow injected instructions
- **`historyTransform` redaction** — strip PII from tool outputs before they re-enter the model context

### App Intents Mitigations
- **`authenticationPolicy`** — require biometric/passcode authentication before an intent executes
- Schema-based default policies — intents using sensitive schemas (e.g., `.photos.deleteAssets`) inherit `requiresAuthentication` by default

### Secure Prompt Design
Keep system instructions clearly separated from user and tool content. Limit the scope of tools: avoid giving an agent that reads untrusted content access to high-privilege irreversible actions in the same session profile.

## APIs & Frameworks

### Foundation Models framework
- `LanguageModelSession` — the core session type for on-device LLM inference
- `LanguageModelSession.DynamicProfile` protocol — composable session configuration
- `Profile` — builder type for assembling instructions, tools, and modifiers
- `Instructions(...)` — sets system prompt within a profile
- `Tool` protocol — defines a callable tool; implement `name`, `description`, and body
- `.model(SystemLanguageModel())` modifier — selects the on-device system model **[NEW]**
- `.onToolCall { call in ... }` modifier **[NEW]** — hook called before tool execution; throw to cancel
- `.historyTransform { entries in ... }` modifier **[NEW]** — transforms the conversation history before it is fed back to the model; used for spotlighting and redaction
- `Transcript.Segment` — the unit operated on inside `historyTransform`
- `Transcript.Entry` — enum of history entries; case `.toolOutput(ToolOutput)` for intercepting tool results
- `ToolOutput.segments` — array of `Transcript.Segment` to rewrite
- `ConfirmationAction.confirmWithUser()` — presents a user-facing confirmation dialog (used in `onToolCall`)

### App Intents framework
- `AppIntent` protocol — base protocol for all intents
- `DeleteIntent` protocol — specialized intent for deletion actions
- `IntentAuthenticationPolicy` enum — `.requiresAuthentication`, `.alwaysAllowed` **[NEW signals]**
- `static var authenticationPolicy: IntentAuthenticationPolicy` — property on an intent struct to declare its policy **[NEW]**
- `@AppIntent(schema:)` attribute — maps an intent to an Apple-defined schema; schema may supply a default authentication policy **[NEW]**
- Schema example: `.photos.deleteAssets` — inherits `.requiresAuthentication` by default

### Security (general)
- Apple Security Overview — referenced for broader security context

## Code Highlights

Require user confirmation before a risky tool executes:
```swift
.onToolCall { call in
    guard call.toolName == "orderTeaTool" else { return }
    guard ConfirmationAction.confirmWithUser() else {
        throw LooseLeafError.userConfirmationDenied
    }
}
```

Spotlight untrusted tool output to reduce injection risk:
```swift
.historyTransform { entries in
    entries.map { entry in
        guard case .toolOutput(var toolOutput) = entry,
              toolOutput.toolName == "postAndFetchPublicFeedTool"
        else { return entry }
        toolOutput.segments = toolOutput.segments.map { segment in
            delimit(segment: segment,
                    startDelimiter: "<<UNTRUSTED>>",
                    endDelimiter: "<</UNTRUSTED>>")
        }
        return .toolOutput(toolOutput)
    }
}
```

Require authentication for a destructive App Intent:
```swift
struct DeletePhotoIntent: DeleteIntent {
    static var authenticationPolicy: IntentAuthenticationPolicy = .requiresAuthentication
    func perform() async throws -> some IntentResult { ... }
}
```

## Takeaways
- Treat any tool that reads untrusted content as a potential injection vector; add `onToolCall` confirmation before any high-risk action it can trigger.
- Use `historyTransform` for spotlighting and redaction to limit how much the model trusts tool outputs.
- Declare `authenticationPolicy = .requiresAuthentication` on intents that perform irreversible or sensitive operations.
- Conduct a formal threat model — list context sources and actions, then connect untrusted inputs to high-risk outputs — before shipping agentic features.

---
_Source: WWDC26 Session 347 page (abstract, chapter summaries, code samples, and resource links)._
