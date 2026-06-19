# Capture Machine-Readable Codes and Text with VisionKit
**WWDC22 · Session 10025** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10025/)

_Platforms:_ iOS 16, iPadOS 16

## Overview
iOS 16 introduces `DataScannerViewController` in the VisionKit framework — a `UIViewController` subclass that encapsulates AVFoundation camera setup and Vision framework text/barcode recognition into a single, easy-to-adopt API. It replaces the need to manually wire together `AVCaptureSession`, `AVCaptureVideoDataOutput`, and Vision recognition requests for live scanning workflows. The scanner provides a live camera preview, guidance labels, system item highlights, tap-to-focus/select, and pinch-to-zoom out of the box.

Supported use cases include any app that needs to scan text or barcodes from a live video feed: pick-and-pack, warehouse management, point-of-sale, and document scanning. The new `DataScannerViewController` complements the `ImageAnalyzer`/`ImageAnalysisInteraction` API (session 10026) which handles Live Text on still images and paused video frames. Supported on 2018 and newer devices with the Apple Neural Engine.

## Key Topics

### DataScannerViewController Setup
- Check `DataScannerViewController.isSupported` before exposing scanning UI — not all devices support it **[NEW]**
- Check `DataScannerViewController.isAvailable` — also validates camera permissions and Screen Time restrictions **[NEW]**
- Initialize with `DataScannerViewController(recognizedDataTypes:qualityLevel:recognizesMultipleItems:isHighFrameRateTrackingEnabled:isPinchToZoomEnabled:isGuidanceEnabled:isHighlightingEnabled:)` **[NEW]**
- Present like any `UIViewController`; call `startScanning()` after presentation completes
- Call `stopScanning()` when done

### RecognizedDataType Configuration
- `.barcode(symbologies: [VNBarcodeSymbology])` — specify barcode types (all Vision symbologies supported)
- `.text()` — any text
- `.text(languages: [String])` — hint the recognizer with expected languages; uses user's preferred languages if omitted
- `.text(textContentType: DataScannerViewController.TextContentType)` — filter to semantic text types: `.URL`, `.phoneNumber`, `.emailAddress`, `.streetAddress`, `.dateTimeDuration`, `.currency`, `.flightNumber`, `.shipmentTrackingNumber` **[NEW]**
- Language support: same as Live Text; iOS 16 adds Japanese and Korean **[NEW]**
- Use `DataScannerViewController.supportedTextRecognitionLanguages` to query current supported list **[NEW]**

### Recognition Quality Levels
- `qualityLevel: .balanced` (default) — good for most cases
- `.fast` — lower resolution, better performance for large easily-legible items (signs, large barcodes)
- `.accurate` — best accuracy for small items (micro QR codes, tiny serial numbers)

### Item Highlights and Region of Interest
- `isHighlightingEnabled: Bool` — enable/disable built-in system highlights; disable to draw custom highlights
- `regionOfInterest: CGRect?` — limit active scanning area (in view coordinates, not image coordinates)
- `overlayContainerView: UIView` — add custom highlight views here; sits above camera preview but below system chrome

### RecognizedItem and Delegate
- `DataScannerViewControllerDelegate` protocol **[NEW]**
- `dataScanner(_:didTapOn:)` — called when user taps a recognized item; receives `RecognizedItem`
- `dataScanner(_:didAdd:allItems:)` — new items entered the scene
- `dataScanner(_:didUpdate:allItems:)` — items moved/changed (transcription can improve over time)
- `dataScanner(_:didRemove:allItems:)` — items left the scene
- `allItems` array in each delegate method is in natural reading order (index 0 is first readable item)
- `RecognizedItem` — enum with `.text(RecognizedItem.Text)` and `.barcode(RecognizedItem.Barcode)` cases **[NEW]**
  - `RecognizedItem.id: RecognizedItem.ID` — unique ID; stable for item's lifetime in the frame
  - `RecognizedItem.bounds: RecognizedItem.Bounds` — four corner points in view coordinates (not a `CGRect`)
  - `RecognizedItem.Text.transcript: String` — recognized text string
  - `RecognizedItem.Barcode.payloadStringValue: String?` — barcode string payload

### AsyncStream Interface
- `DataScannerViewController.recognizedItems: AsyncStream<[RecognizedItem]>` — alternative to delegate; iterate with `for await` **[NEW]**
- Items delivered in natural reading order; use `CollectionDifference` to track adds/removes

### Photo Capture
- `DataScannerViewController.capturePhoto() async throws -> UIImage` — capture a full-resolution still photo while scanning **[NEW]**

## APIs & Frameworks

**VisionKit** (new API) **[NEW]**
- `DataScannerViewController` — `UIViewController` subclass **[NEW]**
  - `class var isSupported: Bool` **[NEW]**
  - `class var isAvailable: Bool` **[NEW]**
  - `class var supportedTextRecognitionLanguages: [String]` **[NEW]**
  - `init(recognizedDataTypes:qualityLevel:recognizesMultipleItems:isHighFrameRateTrackingEnabled:isPinchToZoomEnabled:isGuidanceEnabled:isHighlightingEnabled:)` **[NEW]**
  - `var recognizedDataTypes: Set<RecognizedDataType>`
  - `var qualityLevel: QualityLevel` — `.balanced`, `.fast`, `.accurate`
  - `var recognizesMultipleItems: Bool`
  - `var isHighFrameRateTrackingEnabled: Bool`
  - `var isPinchToZoomEnabled: Bool`
  - `var isGuidanceEnabled: Bool`
  - `var isHighlightingEnabled: Bool`
  - `var regionOfInterest: CGRect?`
  - `var overlayContainerView: UIView` (read-only)
  - `func startScanning() throws` **[NEW]**
  - `func stopScanning()` **[NEW]**
  - `func capturePhoto() async throws -> UIImage` **[NEW]**
  - `var recognizedItems: AsyncStream<[RecognizedItem]>` **[NEW]**
  - `var delegate: DataScannerViewControllerDelegate?`
- `DataScannerViewController.RecognizedDataType` — `.barcode(symbologies:)`, `.text()`, `.text(languages:)`, `.text(textContentType:)` **[NEW]**
- `DataScannerViewController.TextContentType` — `.URL`, `.phoneNumber`, `.emailAddress`, `.streetAddress`, `.dateTimeDuration`, `.currency`, `.flightNumber`, `.shipmentTrackingNumber` **[NEW]**
- `RecognizedItem` — enum **[NEW]**
  - `RecognizedItem.Text` — `.transcript: String`
  - `RecognizedItem.Barcode` — `.payloadStringValue: String?`
  - `.id: RecognizedItem.ID`
  - `.bounds: RecognizedItem.Bounds` (four corner points)
- `DataScannerViewControllerDelegate` **[NEW]**
  - `dataScanner(_:didTapOn:)`
  - `dataScanner(_:didAdd:allItems:)`
  - `dataScanner(_:didUpdate:allItems:)`
  - `dataScanner(_:didRemove:allItems:)`

## Code Highlights

Basic setup and presentation:
```swift
import VisionKit

guard DataScannerViewController.isSupported,
      DataScannerViewController.isAvailable else { return }

let recognizedDataTypes: Set<DataScannerViewController.RecognizedDataType> = [
    .barcode(symbologies: [.qr]),
    .text(textContentType: .URL)
]
let dataScanner = DataScannerViewController(recognizedDataTypes: recognizedDataTypes)
dataScanner.delegate = self
present(dataScanner, animated: true) {
    try? dataScanner.startScanning()
}
```

Handling tap on a recognized item:
```swift
func dataScanner(_ dataScanner: DataScannerViewController, didTapOn item: RecognizedItem) {
    switch item {
    case .text(let text):
        print("text: \(text.transcript)")
    case .barcode(let barcode):
        print("barcode: \(barcode.payloadStringValue ?? "unknown")")
    default:
        break
    }
}
```

Custom highlight views via delegate:
```swift
var itemHighlightViews: [RecognizedItem.ID: HighlightView] = [:]

func dataScanner(_ dataScanner: DataScannerViewController,
                 didAdd addedItems: [RecognizedItem], allItems: [RecognizedItem]) {
    for item in addedItems {
        let view = HighlightView(item: item)
        itemHighlightViews[item.id] = view
        dataScanner.overlayContainerView.addSubview(view)
    }
}

func dataScanner(_ dataScanner: DataScannerViewController,
                 didUpdate updatedItems: [RecognizedItem], allItems: [RecognizedItem]) {
    for item in updatedItems {
        itemHighlightViews[item.id].map { animate($0, to: item.bounds) }
    }
}

func dataScanner(_ dataScanner: DataScannerViewController,
                 didRemove removedItems: [RecognizedItem], allItems: [RecognizedItem]) {
    for item in removedItems {
        itemHighlightViews.removeValue(forKey: item.id)?.removeFromSuperview()
    }
}
```

AsyncStream alternative for tracking items:
```swift
var currentItems: [RecognizedItem] = []
func observeItems() async {
    for await newItems in dataScanner.recognizedItems {
        let diff = newItems.difference(from: currentItems) { $0.id == $1.id }
        if !diff.isEmpty {
            currentItems = newItems
            // Update UI
        }
    }
}
```

## Takeaways
- `DataScannerViewController` replaces complex AVFoundation + Vision wiring with a single `UIViewController` subclass — check `isSupported` and `isAvailable` before showing scanning UI.
- Use `.text(textContentType:)` to filter recognition to semantically meaningful text (URLs, phone numbers, addresses) rather than all raw text.
- For custom highlights, use the `didAdd`/`didUpdate`/`didRemove` delegate trio with `RecognizedItem.id` as the key — item IDs are stable for as long as the item is visible in the frame.
- Use `recognizedItems` AsyncStream as a simpler alternative to the three-method delegate pattern when full lifecycle control is not needed.

---
_Source: WWDC22 Session 10025 page (abstract, transcript, and code samples)._
