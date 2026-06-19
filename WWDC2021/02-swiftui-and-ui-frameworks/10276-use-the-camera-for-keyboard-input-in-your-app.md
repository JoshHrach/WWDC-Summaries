# Use the Camera for Keyboard Input in Your App
**WWDC21 · Session 10276** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10276/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
iOS 15 introduces a new capability that lets users capture text directly from the physical world using the camera — without typing or dictation. This "Text from Camera" feature integrates into standard text fields and text views, allowing users to photograph invoices, signs, flyers, and printed documents to populate app forms instantly.

Developers can enhance this feature by applying content-type filters so the camera focuses only on the relevant category of text (phone numbers, addresses, flight numbers, etc.). They can also add discoverable launcher buttons and even extend non-text controls like `UIImageView` to accept camera-captured input.

The feature requires an iPhone with a Neural Engine running iOS 15 and requires the user's preferred language to be one of seven supported languages. Developers must verify availability before surfacing camera-input affordances to users.

## Key Topics

### Filtering Content with TextContentType and KeyboardType
Setting `textContentType` and `keyboardType` on a `UITextField` or `UITextView` directs the camera to ignore irrelevant text and highlight only matching content. Seven content types support filtering: `telephoneNumber`, `fullStreetAddress`, `URL`, `emailAddress`, `flightNumber` (new in iOS 15), `shipmentTrackingNumber` (new in iOS 15), and `dateTime` (new in iOS 15). Setting `autocorrectionType = .no` on phone number fields also surfaces a quick-access camera button in the keyboard candidate bar.

### Creating Discoverable Launcher Buttons
When the editing menu or candidate bar isn't visible enough, developers can create a `UIAction` using the new `captureTextFromCamera(responder:identifier:)` factory method. The action supplies its own localized title and system image, making it easy to embed in toolbars, menus, or buttons. Before adding launchers, call `canPerformAction(_:withSender:)` to confirm the capability is available on the current device and responder.

### Extending Custom Views with UIKeyInput
Any `UIResponder` that adopts `UIKeyInput` can receive camera-captured text via `insertText(_:)`. Adopting `UITextInputTraits` additionally enables content-type filtering, while adopting the full `UITextInput` protocol enables a live preview of text before insertion. This allows custom controls — such as a captioned `UIImageView` subclass — to participate in camera input without being text controls themselves.

## APIs & Frameworks

**UIKit**
- `UITextField.textContentType` — existing, with new values below
- `UITextField.keyboardType`
- `UITextField.autocorrectionType`
- `UITextContentType.telephoneNumber`
- `UITextContentType.fullStreetAddress`
- `UITextContentType.URL`
- `UITextContentType.emailAddress`
- `UITextContentType.flightNumber` **[NEW]**
- `UITextContentType.shipmentTrackingNumber` **[NEW]**
- `UITextContentType.dateTime` **[NEW]**
- `UIAction.captureTextFromCamera(responder:identifier:)` **[NEW]** — factory method returning a pre-configured `UIAction`
- `UIResponder.captureTextFromCamera(_:)` **[NEW]** — action method invoked by the above `UIAction`
- `UIResponder.canPerformAction(_:withSender:)` — used to gate camera-input affordances
- `UIKeyInput` protocol — `hasText`, `deleteBackward()`, `insertText(_:)`
- `UITextInputTraits` protocol — `keyboardType`, `textContentType`, `autocorrectionType`
- `UITextInput` protocol — `setMarkedText(_:selectedRange:)` (enables insertion preview)
- `UIBarButtonItem(title:image:primaryAction:menu:)`
- `UIMenu(children:)`

## Code Highlights

Setting content types for filtering:
```swift
phone.keyboardType = .phonePad
phone.autocorrectionType = .no
address.textContentType = .fullStreetAddress
```

Creating and embedding a camera-input action in a menu:
```swift
let textFromCamera = UIAction.captureTextFromCamera(responder: self.notes, identifier: nil)
let cameraMenu = UIMenu(children: [choosePhotoOrVideo, takePhotoOrVideo, scanDocuments, textFromCamera])
let menuToolbarItem = UIBarButtonItem(title: nil, image: UIImage(systemName: "camera.badge.ellipsis"),
                                      primaryAction: nil, menu: cameraMenu)
```

Custom `UIImageView` subclass accepting camera input:
```swift
class HeadlineImageView: UIImageView, UIKeyInput {
    var headlineLabel: UILabel = UILabel()
    var hasText: Bool = false

    func insertText(_ text: String) {
        headlineLabel.text = text
    }
    func deleteBackward() { }
}
```

## Takeaways
- Set `textContentType` and `keyboardType` on text fields to enable smart filtering — the camera will highlight only the matching category of text.
- Use `UIAction.captureTextFromCamera(responder:identifier:)` to build discoverable launcher buttons or menu items; always gate them with `canPerformAction(_:withSender:)`.
- Any `UIResponder` that adopts `UIKeyInput` can receive camera-captured text, not just built-in text controls.
- Three new `UITextContentType` values — `flightNumber`, `shipmentTrackingNumber`, and `dateTime` — open camera-input filtering to travel and logistics use cases.

---
_Source: WWDC21 Session 10276 page (abstract, chapter summaries, code samples, and resource links)._
