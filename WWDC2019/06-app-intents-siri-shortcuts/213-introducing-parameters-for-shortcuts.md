# Introducing Parameters for Shortcuts
**WWDC19 · Session 213** · [Watch](https://developer.apple.com/videos/play/wwdc2019/213/)

_Platforms:_ iOS 13, iPadOS 13, watchOS 6 (HomePod, CarPlay, AirPods via Siri)

## Overview
iOS 13 extends the Siri Shortcuts APIs introduced in iOS 12 with support for parameters, enabling two major new capabilities: conversational shortcuts in Siri (multi-turn voice interaction where Siri asks follow-up questions) and user-customizable shortcut actions in the redesigned, now built-in Shortcuts app. Previously, shortcuts had no way for Siri to ask for information at runtime — the shortcut either had all values pre-filled or nothing. Parameters change that fundamentally.

The Shortcuts app is now built into iOS 13 and no longer requires a separate download. It includes a new Automation tab (time- and trigger-based automations), a Gallery tab with curated shortcuts from third-party apps, and a redesigned shortcut editor with a natural-language format that displays parameter values as tappable inline controls. Shortcut actions can now expose outputs (typed results) that can be chained to other actions in multi-step shortcuts.

The session uses the Soup Chef sample app throughout, demonstrating intent definition in Xcode, resolution result types, parameter dependencies, dynamic options, and custom output types.

## Key Topics

**Conversational Shortcuts**
When a user invokes a shortcut via Siri, the new Resolve phase allows the app to interrogate each parameter and ask follow-up questions. Siri calls the intent handler's resolve method for each configurable parameter in declared order. The app returns a `INIntentResolutionResult` subclass to control what Siri does next: ask a question, offer a disambiguation list, confirm the value, report an error, succeed, or skip.

**Resolve Phase (New in iOS 13)**
Three phases now exist: Resolve → Confirm → Handle. Resolve is called per-parameter. The parameter value in the passed intent comes from:
- The user's preset value (if they configured the shortcut in advance).
- The user's spoken input (if Siri just asked a question).
- Empty/nil (if neither).

**Resolution Result Types**
- `needsValue`: Siri asks the custom prompt defined in the Intent editor.
- `disambiguation(with:)`: Siri presents a list for the user to choose from; triggered when a value is ambiguous or options are few.
- `unsupported(forReason:)`: App signals the value is invalid; custom error messages defined per validation error in Xcode.
- `confirmationRequired(with:)`: Siri asks the user to confirm before proceeding.
- `success(with:)`: Parameter is valid; Siri moves to the next parameter.
- `notRequired()`: Parameter is not needed at this time; Siri skips it.

**User Customization in the Shortcuts App**
The same parameter definitions used for voice resolution drive the Shortcuts app editor UI. Users see inline tappable buttons for each parameter value in the natural-language summary. Parameter summaries are authored in the Intent definition file in Xcode and support variable placeholders. The intent must have `isUserConfigurable = true` and each configurable parameter must have `isUserFacing = true`.

**Parameter Dependencies (Parent/Child Relationships)**
Parameters can declare parent/child relationships in the Intent editor. A child parameter is only shown in the Shortcuts app editor when the parent has a specific value. This enables conditional form UI — e.g., show "Delivery Location" only when "Order Type = Delivery", and show "Store Location" only when "Order Type = Pickup". Xcode automatically generates multiple parameter combination summaries for each valid combination.

**Dynamic Options**
Checking "Dynamic Options" for a parameter causes Xcode to generate two additional intent handler protocol methods:
- `provideOptions(for:with:completion:)` — returns the current list of valid choices (e.g., nearby store locations from the server).
- `defaultValue(for:with:completion:)` — returns a pre-selected default value.
When the Resolve method returns `needsValue` and dynamic options are enabled, Siri automatically presents the dynamic list as a disambiguation prompt.

**Outputs and Variables**
Intent responses can now declare custom output properties using new custom types defined in the Intent definition file. Outputs are marked as the designated output of the response. In the Shortcuts app editor, users can tap on the output variable to pick individual properties from the custom type, enabling the output of one action to feed into another action's input. Custom type properties can also be used in Siri response template strings.

**Add to Siri UI Redesign**
The `INUIAddVoiceShortcutButton` sheet is redesigned in iOS 13. Users no longer must record a phrase — they can type or accept a suggested phrase. Tapping "Do" in the sheet opens the parameter customization UI, letting users configure the shortcut from within the app.

**Shortcuts App Changes**
- Built into iOS 13 — no App Store download required.
- New Automation tab: shortcuts that trigger based on time, arrival/departure from locations, car connection, alarm, etc.
- New Gallery tab: curated pre-built shortcuts including a section "Shortcuts from Your Apps" surfacing third-party actions.
- Natural-language editor with inline parameter buttons.
- Action pane: drag-and-drop surface for adding third-party actions.

## APIs & Frameworks

**SiriKit / Intents** (iOS 13) **[NEW or enhanced]**

Intent definition (.intentdefinition file in Xcode):
- `Intent.isUserConfigurable: Bool` — marks intent available to Siri and Shortcuts app **[NEW]**
- `IntentParameter.isUserFacing: Bool` — marks parameter configurable **[NEW]**
- `IntentParameter.displayName: String` — shown in Shortcuts editor **[NEW]**
- Parameter summary string with variable placeholders **[NEW]**
- Parameter relationship: parent/child with `parentParameter` and `parentParameterValues` **[NEW]**
- `IntentParameter.dynamicOptions: Bool` — enables dynamic option methods **[NEW]**
- Custom intent parameter types (beyond system types and enums) **[NEW]**
- Intent response output designation **[NEW]**
- Custom output types (`INIntentResponse` properties of custom types) **[NEW]**

Intent handler protocol (Xcode-generated):
- `resolve<Parameter>(for intent:completion:)` — per-parameter resolve method **[NEW]**
- `provide<Parameter>Options(for intent:completion:)` — dynamic options **[NEW]**
- `default<Parameter>(for intent:completion:)` — default value for dynamic options **[NEW]**
- `confirm(intent:completion:)` (existing)
- `handle(intent:completion:)` (existing)

Resolution result types:
- `INIntentResolutionResult.needsValue()` **[NEW]**
- `INIntentResolutionResult.disambiguation(with:)` **[NEW — enhanced]**
- `INIntentResolutionResult.unsupported(forReason:)` **[NEW]**
- `INIntentResolutionResult.confirmationRequired(with:)` **[NEW — enhanced]**
- `INIntentResolutionResult.success(with:)` (existing)
- `INIntentResolutionResult.notRequired()` **[NEW]**

Add to Siri:
- `INUIAddVoiceShortcutButton` — redesigned sheet, no phrase recording required **[NEW design]**
- `INUIAddVoiceShortcutViewController` / `INUIEditVoiceShortcutViewController` (existing)

Automation (new system feature, no public API required for basic triggers):
- Shortcuts automation triggers (time, location, CarPlay, alarm, etc.) **[NEW user feature]**

**Xcode 11**
- Intent definition file editor — parameter summary editor, parent/child relationships, dynamic options, custom types, output designation **[NEW UI]**

## Code Highlights

Resolve method implementation (auto-generated protocol):
```swift
func resolveSoup(for intent: OrderSoupIntent,
                 with completion: @escaping (SoupResolutionResult) -> Void) {
    guard let soup = intent.soup else {
        // No value set — ask Siri to prompt the user
        completion(.needsValue())
        return
    }
    
    guard menuManager.menuItems.contains(soup) else {
        // Item not on the menu
        completion(.unsupported(forReason: .notOnMenu))
        return
    }
    
    // Valid value — proceed to next parameter
    completion(.success(with: soup))
}
```

Providing dynamic options for store location:
```swift
func provideStoreLocationOptions(for intent: OrderSoupIntent,
                                 with completion: @escaping ([StoreLocation]?, Error?) -> Void) {
    storeManager.fetchNearbyStores { stores, error in
        completion(stores, error)
    }
}
```

Disambiguation for ambiguous input:
```swift
func resolveSoup(for intent: OrderSoupIntent,
                 with completion: @escaping (SoupResolutionResult) -> Void) {
    let matches = menuManager.soups(matching: intent.soup)
    if matches.count > 1 {
        completion(.disambiguation(with: matches))
    } else if matches.count == 1 {
        completion(.success(with: matches[0]))
    } else {
        completion(.unsupported(forReason: .notOnMenu))
    }
}
```

## Takeaways
- Parameters bring true conversational AI to Siri Shortcuts — apps now control a multi-turn dialogue by declaring parameters, defining their resolution logic, and customizing Siri's prompts in the Intent editor.
- The same parameter definitions power both the Siri voice conversation and the Shortcuts app editor UI, meaning there is a single source of truth for intent structure.
- Parent/child parameter relationships enable conditional form rendering in the Shortcuts app without any code — all driven by the Intent definition file in Xcode.
- Outputs and custom types open the door to powerful multi-app shortcut chains where the result of one action becomes the input to another directly from the Shortcuts editor.

---
_Source: WWDC19 Session 213 page (abstract, chapter summaries, code samples, and resource links)._
