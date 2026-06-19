# Meet the Contact Access Button
**WWDC24 · Session 10121** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10121/)

_Platforms:_ iOS 18, iPadOS 18

## Overview
iOS 18 introduces a new Limited Access mode for Contacts alongside the new `ContactAccessButton` UI component. Instead of requiring full access to a user's entire contact list, apps can now request access to individual contacts on demand — right at the point of use, with no trip to Settings. This session explains the new access model, how to integrate `ContactAccessButton`, how to fetch contacts after access is granted, and when to use the alternative `contactAccessPicker` sheet.

## Key Topics

**Limited Access Mode**
- Before iOS 18: the Contacts permission was binary — full access or none
- iOS 18 adds a new `CNAuthorizationStatus.limited` case — users can grant access to specific contacts only
- When the app first requests access, users can choose Full Access, Limited Access (select contacts), or Deny
- Later in Settings, users can manage which contacts are shared with the app
- Apps should handle `.limited` status just like `.authorized` — the framework filters the visible contact store automatically

**Contact Access Button**
- `ContactAccessButton` — new SwiftUI view; looks like a search result row with the contact's name, image, and a small "+Add" affordance
- Displays contacts matching a `queryString` (live search string) from the app's Limited contact store
- When the user taps it, the system automatically grants access to that specific contact and returns the contact's `CNContact` identifier
- No additional permission prompt is shown — the tap itself is the authorization act
- The button only shows contacts not already accessible to the app

**Appearance Customization**
- Supports standard SwiftUI modifiers: `.font()`, `.foregroundStyle()`, `.tint()`, `ContactAccessButton.Style` for different visual variants
- Designed to blend into existing search/list UIs

**Accessing Contacts After Authorization**
- After the user taps `ContactAccessButton`, handle the returned identifier in a closure
- Fetch the full `CNContact` using `CNContactStore` with `CNContactFetchRequest` and the contact's identifier
- `CNContactFormatter.descriptorForRequiredKeys(for: .fullName)` — specify which fields to fetch
- Works the same for both `.authorized` (full) and `.limited` access — the store automatically scopes what's visible

**contactAccessPicker (Alternative)**
- `contactAccessPicker(isPresented:)` — SwiftUI sheet modifier; presents a system picker for selecting contacts to share with the app
- Appropriate when `ContactAccessButton` doesn't fit the UI (e.g., no search field, or a settings-style flow)
- Call when the user explicitly opts in; returns an array of newly-authorized contact identifiers

**Which API to Use**
- Use `ContactAccessButton` when your app has a search interface — it's the most contextual and seamless
- Use `.contactAccessPicker` when you need a batch selection flow (e.g., "choose contacts to sync")
- Never ask for full access just to show one contact — Limited Access + these new APIs is the privacy-respecting approach

## APIs & Frameworks

**ContactsUI**
- `ContactAccessButton` **[NEW]** — SwiftUI view that displays a contact matching a query and grants access on tap
- `ContactAccessButton(queryString:)` **[NEW]** — initializer; takes a `@Binding String` live search query
- `ContactAccessButton.Style` **[NEW]** — visual style variants for the button
- `.contactAccessPicker(isPresented:)` **[NEW]** — SwiftUI view modifier; presents a contact selection sheet

**Contacts**
- `CNAuthorizationStatus.limited` **[NEW]** — new authorization status case for iOS 18 Limited Access mode
- `CNContactStore` — existing API; scoped automatically to accessible contacts in Limited mode
- `CNContactFetchRequest` — fetch contacts by identifier with specified key descriptors
- `CNContactFormatter.descriptorForRequiredKeys(for:)` — specify fields needed (e.g., `.fullName`)
- `CNContact` — contact data object

## Code Highlights

Using `ContactAccessButton` in a search result list:
```swift
@Binding var searchText: String
@State var authorizationStatus: CNAuthorizationStatus = .notDetermined

var body: some View {
    List {
        ForEach(searchResults(for: searchText)) { result in
            ResultRow(result)
        }
        ContactAccessButton(queryString: searchText) { identifier in
            let contacts = await fetchContacts(withIdentifiers: [identifier])
            // show the newly-accessible contact
        }
    }
}
```

Fetching a contact by identifier:
```swift
func fetchContacts(withIdentifiers identifiers: [String]) async -> [CNContact] {
    return await Task {
        let keys = [CNContactFormatter.descriptorForRequiredKeys(for: .fullName)]
        let request = CNContactFetchRequest(keysToFetch: keys)
        var results = [CNContact]()
        try? CNContactStore().enumerateContacts(with: request) { contact, _ in
            results.append(contact)
        }
        return results
    }.value
}
```

Using the contact access picker sheet:
```swift
@State private var isPresented = false

Button("Show picker") { isPresented.toggle() }
    .contactAccessPicker(isPresented: $isPresented) { identifiers in
        let contacts = await fetchContacts(withIdentifiers: identifiers)
        // use the new contacts!
    }
```

## Takeaways
- Adopt `ContactAccessButton` in search interfaces to let users grant access to individual contacts at the exact moment they need them — no upfront permission dialog required.
- Request Limited Access (not full access) by default — `CNContactStore` automatically filters what's visible and your existing fetch code works unchanged.
- Use `.contactAccessPicker` as a fallback for flows that don't have a search field or need batch selection.
- Handle `CNAuthorizationStatus.limited` the same way as `.authorized` — the system scopes the contact store for you.

---
_Source: WWDC24 Session 10121 page (abstract, chapter list, code samples, and resource links)._
