# Unlock In-Game Content with StoreKit and Background Assets
**WWDC26 · Session 378** · [Watch](https://developer.apple.com/videos/play/wwdc2026/378/)

_Platforms:_ iOS, iPadOS, macOS, tvOS (game development via Unity; Background Assets and StoreKit native frameworks)

## Overview
This session focuses on Apple platform game developers, particularly those shipping through Unity, and delivers three major improvements. First, Apple-Hosted Background Assets now support **localized asset packs** — players automatically receive only the language-specific assets they need, dramatically shrinking effective download sizes. Second, a new **Steam Asset Converter** (`xcrun ba-package convert`) makes it straightforward to translate existing Steam depot manifests into Background Assets asset pack manifests. Third, two new **Apple Unity plug-ins** — for StoreKit and for Background Assets — expose native Swift framework APIs as C# types, enabling Unity games to use Apple's native In-App Purchase and on-demand asset delivery without bridging boilerplate.

The session also briefly covers the App Store and Apple Games app visual presence improvements (new product page header and search result placements introduced in Session 205) as they apply to the games context.

## Key Topics

### Background Assets: Apple-Hosted and Localized Asset Packs (0:33–3:14)
- **Apple-Hosted Managed Background Assets**: up to 200 GB per app is included in the Apple Developer Program at no extra charge.
- Players download only the asset packs they actually need, reducing storage footprint.
- **Localized asset packs** (new): a `language` key in the asset pack manifest tells the system which language the pack covers; the OS automatically selects and downloads packs matching the player's language preference set in Settings.
- This is particularly impactful for games with large per-language voice-over or localised art assets.

### Convert Steam Depots to Asset Packs (3:14–4:15)
- A new **Steam Asset Converter** (`xcrun ba-package convert`) takes a Steam `.vdf` depot file and produces a Background Assets JSON manifest.
- The manifest is then packaged into a `.aar` asset pack archive using `xcrun ba-package`.
- Two commands complete the migration: `convert` to generate the manifest, then `ba-package` to build the archive.

### Unity Plug-ins (4:15–5:52)
- Two new **Apple Unity plug-ins** available on GitHub (`github.com/apple/unityplugins`):
  - **Apple.StoreKit** — C# bindings for the native StoreKit 2 framework
  - **Apple.BackgroundAssets** — C# bindings for the native Background Assets framework
- Requirements: Xcode 27, Python 3, Unity 2022 LTS or later.
- Plug-in APIs mirror the Swift API surface closely, including `async`/`await` patterns via Unity's C# task system.

### StoreKit and Background Assets Sample Code (5:52–8:25)
- `Product.FetchProducts(string[])` — async fetch of products by ID.
- `product.Purchase()` — returns a result with `PurchaseResult.ResultEnum` and a `TransactionVerification` containing `IsVerified` and `SafePayload`.
- `Transaction.Updates` — static event; subscribe to receive `VerificationResult<Transaction>` for all transaction updates (including consumable revocations).
- `Transaction.GetCurrentEntitlements()` — async enumerable; source of truth for non-consumable and subscription entitlements.
- `AssetPackManager.GetManifestAsync()` — fetches the `AssetPackManifest`.
- `AssetPackManager.EnsureLocalAvailabilityOfAssetPackAsync(AssetPack)` — triggers download and waits for completion.
- `AssetPackManager.DownloadStatusUpdatesAsync(string)` — async enumerable of `DownloadStatusUpdate` for progress UI.

### Game Presence (8:25)
- New product page header and search result visual placements (introduced in Session 205) apply fully to games in both the App Store and the Apple Games app.
- Upload brand-quality images and videos to Asset Library in App Store Connect to take advantage.

## APIs & Frameworks

### Background Assets (Native — Apple Framework)
- **`BackgroundAssets`** framework — `https://developer.apple.com/documentation/BackgroundAssets`
- **Asset pack manifest** JSON format — `assetPackID`, `downloadPolicy`, `language` **[NEW]**, `sourceRoot`, `fileSelectors`, `platforms`
- **`language` key in asset pack manifest** **[NEW]** — BCP 47 locale tag (e.g. `"en-US"`) enabling language-driven selective delivery
- Apple-Hosted Background Assets — up to 200 GB per app included in Developer Program **[NEW capacity]**

### Background Assets — Unity Plug-in (NEW)
- **`Apple.BackgroundAssets`** C# namespace **[NEW]**
- `AssetPackManager.GetManifestAsync()` → `AssetPackManifest`
- `AssetPackManifest.GetAssetPack(string assetPackId)` → `AssetPack`
- `AssetPackManager.EnsureLocalAvailabilityOfAssetPackAsync(AssetPack)` — awaitable download
- `AssetPackManager.DownloadStatusUpdatesAsync(string)` → `IAsyncEnumerable<AssetPackManager.DownloadStatusUpdate>`

### StoreKit — Unity Plug-in (NEW)
- **`Apple.StoreKit`** C# namespace **[NEW]**
- `Product.FetchProducts(string[])` → `Product[]`
- `product.Purchase()` → `PurchaseResultWrapper` containing `PurchaseResult.ResultEnum` and `TransactionVerification`
- `TransactionVerification.IsVerified` — bool
- `TransactionVerification.SafePayload` — verified `Transaction`
- `Transaction.SafePayload.Finish()` — finish a verified transaction
- `Transaction.Updates` — static `event Action<VerificationResult<Transaction>>`
- `Transaction.GetCurrentEntitlements()` → `IAsyncEnumerable<VerificationResult<Transaction>>`
- `ProductType.ProductTypeEnum` — `Consumable`, `NonConsumable`, `AutoRenewableSubscription`
- `Transaction.RevocationDate` — non-null indicates a revoked consumable

### Steam Asset Converter (xcrun tools — NEW)
- `xcrun ba-package convert --asset-pack-id <id> --l <locale> --on-demand <depot.vdf> -o <manifest.json>`
- `xcrun ba-package <manifest.json> -o <archive.aar>`

### StoreKit (Native Swift)
- **`StoreKit`** framework — `https://developer.apple.com/documentation/StoreKit`

## Code Highlights

Fetch and purchase with the Unity StoreKit plug-in:
```csharp
var products = await Product.FetchProducts(new[] { "com.thecoast.capecod" });
var result = await product.Purchase();
if (result.Result == PurchaseResult.ResultEnum.Success && result.TransactionVerification.IsVerified) {
    result.TransactionVerification.SafePayload.Finish();
}
```

Listen for transaction updates (handles consumable revocation):
```csharp
Transaction.Updates += async (VerificationResult<Transaction> result) => {
    if (!result.IsVerified) return;
    var tx = result.SafePayload;
    if (tx.ProductType == ProductType.ProductTypeEnum.Consumable) {
        if (tx.RevocationDate != null) { /* revoke */ } else { /* grant */ }
    } else {
        await foreach (var r in Transaction.GetCurrentEntitlements()) { /* grant */ }
    }
    tx.Finish();
};
```

Convert a Steam depot and package it:
```shell
xcrun ba-package convert --asset-pack-id voice-english --l en-US --on-demand voice-english.vdf -o voice-english.json
xcrun ba-package voice-english.json -o voice-english.aar
```

## Takeaways
- **Localized asset packs** with a `language` field in the manifest enable the OS to deliver only the player's language assets, significantly reducing effective download and storage sizes.
- The **Steam Asset Converter** (`xcrun ba-package convert`) makes it practical to migrate existing Steam depot structures to Apple Background Assets with minimal manual work.
- Two new **Apple Unity plug-ins** (StoreKit and Background Assets) give Unity game developers native-quality IAP and asset delivery in C# without writing any Swift bridge code; they require Unity 2022 LTS and Xcode 27.
- Up to **200 GB of Apple-Hosted Background Assets** is now included in the Apple Developer Program, removing cost concerns for large game asset bundles.

---
_Source: WWDC26 Session 378 page (abstract, chapter summaries, code samples, and resource links)._
