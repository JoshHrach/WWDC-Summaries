# What's New in Wallet
**WWDC26 · Session 209** · [Watch](https://developer.apple.com/videos/play/wwdc2026/209/)

_Platforms:_ iOS 27+, watchOS (Wallet passes); macOS (Pass Designer, Pass Builder server tool); cross-platform server-side pass generation

## Overview
This session delivers the most significant update to the Apple Wallet pass developer experience in several years, touching the visual design language, the barcode format library, a new pass interaction model, and a completely new developer toolchain. On the design side, iOS 27 introduces **Poster Generic** — a new pass style that leads with a large background image and puts branding and key fields in a modern poster-like layout. Four new barcode formats join the existing set, addressing industries (healthcare, retail, ticketing) that rely on legacy scanners.

The standout toolchain additions are **Pass Designer** — a Mac app providing a true WYSIWYG pass editor — and **Pass Builder** — a Swift package (also usable from Java via swift-java and from any language via protobuf definitions) for server-side personalisation, signing, and distribution of passes at scale. Together these tools dramatically lower the barrier for both prototyping new pass designs and running production pass-issuance infrastructure.

## Key Topics

### Poster Generic — New Pass Style (0:40–2:36)
- **Poster Generic** is a brand-new pass style in iOS 27 (`"posterGeneric"` key in `pass.json`).
- Layout: full-bleed background image, primary logo, header fields, primary fields, a footer field, and an optional barcode.
- Designed for membership cards, loyalty cards, and event tickets that benefit from rich visual branding.
- For **backwards compatibility on iOS 26 and earlier**, include both `"posterGeneric"` and `"generic"` keys in `pass.json`; older OS versions use the `generic` style, iOS 27 uses `posterGeneric`.

### Barcodes — New Formats (2:36–4:27)
- iOS 27 adds four new barcode format constants to the `barcodes` array in `pass.json`:
  - **`PKBarcodeFormatCodabar`** — used in healthcare, libraries, blood banks
  - **`PKBarcodeFormatCode39`** — industrial and logistics
  - **`PKBarcodeFormatEAN13`** — retail point-of-sale
  - **`PKBarcodeFormatITF`** (Interleaved 2 of 5) — cartons, warehousing
- Specified using the same existing `Barcode` object with a `format` field.
- For **backward compatibility**, add a widely supported fallback barcode (e.g. `PKBarcodeFormatQR`) as a second entry in the `barcodes` array.

### Featured Actions (4:27–5:46)
- **Featured actions** are a new top-level key (`"featuredActions"`) in `pass.json` available in iOS 27.
- Available for **all pass styles** (not just specific types).
- Each action is an object with: `identifier` (unique string), `type` (predefined action type string, e.g. `"membershipBenefits"`, `"viewMembership"`), and `url`.
- Up to **2 featured actions** per pass, listed in priority order.
- Actions surface prominently in the pass UI, giving users quick access to relevant deep-links or web content.

### Pass Designer (5:47–10:40)
- **Pass Designer** is a new **Mac application** providing a WYSIWYG pass editor.
- Renders passes exactly as they appear on iOS, giving designers real-time visual feedback.
- Supports all existing pass styles and the new Poster Generic style.
- Outputs a **`.pkpasstemplate`** file — a reusable template that captures the pass structure, field keys, default values, and assets.
- Templates are the input format for Pass Builder.

### Pass Builder (10:40–13:50)
- **Pass Builder** is a new **Swift package** for server-side pass personalisation and signing.
- Add as a Swift Package Manager dependency; exposes a clean Swift API.
- Core types:
  - `PassPackage(url:)` — loads a `.pkpasstemplate`
  - `package.pass.fields.setValue(_:forKey:)` — substitutes values for template field keys
  - `PassImage(url:)` — wraps an image asset for assignment to `package.background`, etc.
  - `Pass.Barcode(message:format:)` — constructs a barcode
  - `Pass.Action(id:type:url:)` — constructs a featured action
  - `PassCertificate(url:password:)` — loads a `.p12` pass certificate
  - `PassSigner(passCertificate:wwdrCertifiate:)` — signs and writes a `.pkpass` file
  - `signer.signPass(_:writingTo:)` — finalises and writes the signed pass to disk
- **Cross-language support**:
  - **Java**: the `swift-java` project generates native Java bindings for the Swift API, callable from the JVM.
  - **Other languages**: Apple is publishing **protobuf definitions** of the Pass Package format so teams can generate type-safe models in any language, then invoke the `buildpass` command-line executable to personalise and sign.

### Personalizing a Pass Template (13:50)
- Template field keys (e.g. `DOG_NAME`, `LOVES`, `MEMBER_ID`) are defined in Pass Designer and substituted at runtime in Pass Builder.
- Protobuf-based workflow: generate a customisation message from any language, serialise to protobuf, invoke `buildpass` CLI with the template and customisation message to produce a signed `.pkpass`.

## APIs & Frameworks

### PassKit / Wallet — pass.json Keys
- **`"posterGeneric"`** **[NEW]** — new top-level pass style key for iOS 27; structured like `generic` with `headerFields` and `footerFields`
- **`"generic"`** — existing pass style; used as backward-compatible fallback alongside `posterGeneric`
- **`"featuredActions"`** **[NEW]** — top-level array of `Action` objects; up to 2 actions; available for all pass styles
  - Action object: `identifier` (string), `type` (string), `url` (string)
  - Known action types: `"membershipBenefits"`, `"viewMembership"`
- **`"barcodes"`** — existing array; now supports four additional `format` values **[NEW]**:
  - `"PKBarcodeFormatCodabar"` **[NEW]**
  - `"PKBarcodeFormatCode39"` **[NEW]**
  - `"PKBarcodeFormatEAN13"` **[NEW]**
  - `"PKBarcodeFormatITF"` **[NEW]**

### Pass Builder Swift Package (NEW)
- **`PassPackage`** — `init(url: URL)` loads a `.pkpasstemplate`
- **`PassPackage.pass.fields.setValue(_:forKey:)`** — template variable substitution
- **`PassPackage.background`** — `PassImage` property for the pass background
- **`PassPackage.featuredActions`** — `[Pass.Action]`
- **`Pass.Barcode`** — `init(message: String, format: BarcodeFormat)`
- **`Pass.Action`** — `init(id: String, type: String, url: URL)`
- **`PassImage`** — `init(url: URL)`
- **`PassCertificate`** — `init(url: URL, password: String?)` — loads `.p12` or `.cer` certificate files
- **`PassSigner`** — `init(passCertificate: PassCertificate, wwdrCertifiate: PassCertificate)`
- **`PassSigner.signPass(_:writingTo:)`** — produces signed `.pkpass` at destination URL
- **`buildpass`** CLI **[NEW]** — command-line executable for non-Swift language workflows

### Pass Designer (Mac App — NEW)
- WYSIWYG editor; outputs `.pkpasstemplate` files
- True-to-iOS rendering, supports Poster Generic and all existing pass styles

### Cross-Language Integration
- **swift-java** — generates native Java bindings for Pass Builder Swift API **[NEW]**
- **Protobuf definitions** for Pass Package format **[NEW]** — enables type-safe model generation in any language

### PassKit Documentation
- `https://developer.apple.com/documentation/PassKit/wallet`

## Code Highlights

Poster Generic pass.json structure (with Generic fallback):
```json
"posterGeneric": {
  "headerFields": [{ "key": "memberID", "label": "Guest No.", "value": "102035" }],
  "footerFields": [{ "key": "membershipType", "value": "Family Pass" }]
},
"generic": {
  "headerFields": [{ "key": "memberID", "label": "Guest No.", "value": "102035" }],
  "footerFields": [{ "key": "membershipType", "value": "Family Pass" }]
}
```

New barcode format with QR fallback for older OS:
```json
"barcodes": [
  { "format": "PKBarcodeFormatCodabar", "message": "123456789", "messageEncoding": "iso-8859-1" },
  { "format": "PKBarcodeFormatQR",      "message": "123456789", "messageEncoding": "iso-8859-1" }
]
```

Server-side pass personalisation with Pass Builder:
```swift
var package = PassPackage(url: "template.pkpasstemplate")
package.pass.fields.setValue(member.name, forKey: "MEMBER_NAME")
package.background = PassImage(url: member.photoURL)
package.pass.barcodes = [Pass.Barcode(message: member.id, format: .pdf417)]
package.featuredActions = [Pass.Action(id: "a1", type: "viewMembership", url: member.url)]

let signer = PassSigner(
    passCertificate: try PassCertificate(url: "pass.p12", password: "s3cr3t"),
    wwdrCertifiate:  try PassCertificate(url: "wwdr.cer")
)
try signer.signPass(package, writingTo: destinationURL)
```

## Takeaways
- **Poster Generic** is a new full-bleed, image-led pass style in iOS 27; include both `"posterGeneric"` and `"generic"` keys in `pass.json` for forward and backward compatibility.
- Four new barcode formats (`Codabar`, `Code39`, `EAN-13`, `ITF`) expand Wallet pass compatibility to industries relying on non-QR scanners; always include a QR/PDF417 fallback for older OS versions.
- **Featured actions** expose up to two contextual deep-links (by URL) directly on the pass face for all pass styles, replacing the need for users to hunt through pass details for related links.
- **Pass Designer** (Mac WYSIWYG editor) and **Pass Builder** (Swift package + CLI) form a complete new toolchain for designing and generating signed passes at scale, with cross-language support via swift-java and protobuf definitions.

---
_Source: WWDC26 Session 209 page (abstract, chapter summaries, code samples, and resource links)._
