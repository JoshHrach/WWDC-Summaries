# Building Great Shortcuts
**WWDC19 · Session 805** · [Watch](https://developer.apple.com/videos/play/wwdc2019/805/)

_Platforms:_ iOS 13, iPadOS 13, watchOS 6

## Overview
This session is a practical engineering guide for building high-quality Shortcuts actions using SiriKit intents. It covers three areas: the anatomy of a Shortcuts action (parameters, parameter summaries, display names), the mechanisms for surfacing your app's shortcuts to users (Add to Siri, the Shortcuts Gallery, and the Shortcuts Editor), and the new iOS 13 input/output system that lets actions from different apps chain together in multi-step shortcuts.

The session uses "Soup Chef" (Apple's sample app) as the primary example throughout, demonstrating how to configure an ordering intent as an action, donate interactions so the system learns user habits, and define output types so that one app's result becomes the input of another app's action.

## Key Topics

**Anatomy of a Shortcuts Action**
- Every action corresponds to a custom intent defined in an `.intentdefinition` file in Xcode
- Parameters have a **display name** (shown as placeholder text until user fills in a value; always capitalize)
- **Parameter summary** — sentence-form description shown in the action card; must start with a verb, must not include the app name (shown automatically), should only include required parameters; remaining parameters collapse under "Show More"
- Multiple summaries supported — the displayed summary updates dynamically as the user changes parameter values (e.g., separate summaries for "pickup" vs "delivery")
- Parameter summaries are configured in the Shortcuts App section of the Xcode Intent editor

**Add to Siri — In-App Shortcut Creation**
- Present `INUIAddVoiceShortcutViewController` immediately after the user performs a repeatable action (e.g., placing an order)
- Pre-fill the intent with full information about what the user just did — minimizes follow-up questions during future runs
- Set `INIntent.suggestedInvocationPhrase` — short, speakable, memorable phrase pre-populated in the "Add to Siri" view; user can type or dictate instead of speaking it
- The "Do" section in the view shows a live preview of the shortcut's action; if the intent is configurable, the user can tap it to edit values before saving
- API unchanged from iOS 12; view controller behavior expanded in iOS 13

**Shortcuts Gallery — Discovery**
- iOS 13 adds a new suggested shortcuts section to the Gallery featuring apps the user frequently uses
- Your app controls suggestions in two ways:
  1. **Explicit list**: call `INVoiceShortcutCenter.shared.setShortcutSuggestions(_:)` with an array of `INShortcut` objects; update as user habits change
  2. **Donations**: donate `INInteraction` objects whenever the user performs an action; the system uses these for Gallery suggestions AND private on-device Siri suggestions (Lock Screen, Spotlight)
- Donating: create `INInteraction(intent:response:)`, call `interaction.donate(completion:)`
- Suggestions in the Gallery show the **key parameter** value and image; choose the parameter that is most identifiable to the user as the key parameter in the Intent editor
- Always include an image on the key parameter's `INObject` when donating; falls back to app icon if missing

**Shortcuts Editor — In-Editor Discovery**
- Suggested actions list in the Shortcuts editor is populated from device usage including third-party app donations
- Tapping an app's name in the editor shows all of the app's actions (donated or not)

**Input and Output — Chaining Actions**
- iOS 13 allows actions to **output** a typed object for use as input to subsequent actions **[NEW]**
- Workflow: define a custom type in the Intent editor → add it as a property to the intent's Response → select it from the Response's Output dropdown
- Only one property can be selected as output (control which information downstream actions see)
- **Input parameter**: designate one parameter as the input parameter in the Intent editor so that when the user adds a second action, the output of the first is automatically filled into the input of the second — no manual selection required

## APIs & Frameworks

### SiriKit / Intents Framework
- `INIntent` — base class for custom intents; set `suggestedInvocationPhrase`
- `INIntentResponse` — result object returned by the intent handler; add output property here
- `INInteraction(intent:response:)` — wraps an intent + response for donation
- `INInteraction.donate(completion:)` — donates to the system for suggestions and Siri
- `INVoiceShortcutCenter.shared.setShortcutSuggestions(_:)` **[NEW in iOS 12, reinforced in iOS 13]** — explicitly set Gallery suggestions
- `INShortcut(intent:)` — wraps an intent as a shortcut for suggestion
- `INObject` — typed object used as a key parameter value; supports `identifier`, `displayString`, and `displayImage`

### Intents UI / SiriKit UI
- `INUIAddVoiceShortcutViewController` — modal view controller for in-app "Add to Siri" shortcut creation; unchanged API since iOS 12
- `INUIAddVoiceShortcutButton` — pre-built button that presents `INUIAddVoiceShortcutViewController`

### Xcode Intent Editor
- `.intentdefinition` file — defines intents, parameters, response types, and custom types
- Parameter configuration: display name, type, input parameter flag, key parameter flag
- Parameter summary editor: Shortcuts App section → define multiple summary strings with parameter tokens
- Custom types: define via "+" in the Types section; properties become accessible to downstream actions
- Response Output dropdown: selects which response property becomes the action's output

## Code Highlights

Donating an interaction after a user action:
```swift
let intent = OrderSoupIntent()
intent.soup = INObject(identifier: "tomato", displayString: "Tomato Soup")
intent.quantity = 1
intent.suggestedInvocationPhrase = "Order tomato soup"

let response = OrderSoupIntentResponse(code: .success, userActivity: nil)
let interaction = INInteraction(intent: intent, response: response)
interaction.donate { error in
    if let error = error {
        print("Donation failed: \(error)")
    }
}
```

Explicitly setting Gallery suggestions:
```swift
let soups: [INShortcut] = soupMenu.map { soup in
    let intent = OrderSoupIntent()
    intent.soup = INObject(identifier: soup.id, displayString: soup.name,
                           pronunciationHint: soup.name)
    intent.suggestedInvocationPhrase = "Order \(soup.name)"
    return INShortcut(intent: intent)
}
INVoiceShortcutCenter.shared.setShortcutSuggestions(soups)
```

Presenting Add to Siri after a successful order:
```swift
let intent = OrderSoupIntent()
intent.soup = order.soupItem
intent.quantity = order.quantity as NSNumber
intent.suggestedInvocationPhrase = "Order \(order.soupItem.displayString)"

let shortcut = INShortcut(intent: intent)
let viewController = INUIAddVoiceShortcutViewController(shortcut: shortcut)
viewController.delegate = self
present(viewController, animated: true)
```

## Takeaways
- Place the "Add to Siri" button immediately after the user completes a repeatable action — that is the moment of highest intent to repeat and the best time to capture a shortcut.
- Pre-fill intents with complete action data and set `suggestedInvocationPhrase` — this reduces follow-up questions during future Siri runs and improves the shortcut preview in the Add to Siri view controller.
- Donate consistently: donations power Gallery suggestions, Spotlight suggestions, Lock Screen suggestions, and the Shortcuts Editor's suggested actions list simultaneously.
- Select a meaningful key parameter and always provide an image for it; without an image the system falls back to the app icon in suggestion lists.
- Define output types on your intents' responses so that your actions become composable building blocks in multi-step shortcuts — input/output chaining is the primary way power users combine your app's actions with other apps.

---
_Source: WWDC19 Session 805 page (abstract and full transcript)._
