# One-tap account security upgrades
**WWDC20 · Session 10666** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10666/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
iOS 14 introduces two related capabilities for improving account security. First, iCloud Keychain's Security Recommendations section now identifies passwords involved in known data breaches and sends users proactive notifications. Second, the new **Account Authentication Modification Extension** point allows apps to offer one-tap account upgrades directly from the password manager or from within the app itself — either switching to Sign in with Apple or replacing a weak/breached password with a strong, system-generated one.

The extension is a `UIViewController` subclass from `AuthenticationServices`. It runs out-of-process, communicates with the app's backend server, and optionally displays security step-up UI (e.g., for two-factor authentication) before committing the account change.

## Key Topics

### iCloud Keychain Security Recommendations (New)
- New section in Settings > Passwords that flags: reused passwords, easily guessed passwords, and (new in iOS 14) passwords seen in public data breaches
- High-severity issues (breach) are highlighted and trigger push notifications directing users to the password manager
- Each warning includes a "Change Password on Website" button that navigates to the service's `.well-known/change-password` URL (standard from IETF)
- New in iOS 14: one-tap upgrade buttons powered by the Account Authentication Modification Extension

### Account Authentication Modification Extension
- App Extension target created with Xcode's Account Authentication Modification Extension template
- Subclasses `ASAccountAuthenticationModificationViewController` **[NEW]** — a `UIViewController`
- Two upgrade types (each declared independently in Info.plist):
  - **Strong password upgrade** (`ASAccountAuthenticationModificationSupportStrongPasswordChange: YES`)
  - **Sign in with Apple upgrade** (`ASAccountAuthenticationModificationSupportsUpgradeToSignInWithApple: YES`)
- Accessible from three places: password detail view, Security Recommendations item, and automatic system prompt when user signs in with a weak/breached password
- Also invocable programmatically from the app via `ASAccountAuthenticationModificationController` **[NEW]**

### Strong Password Upgrade Flow (No Step-Up UI)
1. System calls `changePasswordWithoutUserInteraction(for:existingCredential:newPassword:userInfo:)` — passes the domain/URL, current credential, and a system-generated strong password
2. Extension authenticates with backend using existing cookies/tokens or the passed credential
3. On success: commit password change on server, call `extensionContext.completeChangePasswordRequest(updatedCredential:userInfo:)` with the new `ASPasswordCredential`
4. On failure: call `extensionContext.cancelRequest(withError:)` with `ASAccountAuthenticationModificationErrorCode.failed`; optionally set `ASExtensionLocalizedFailureReasonErrorKey` in the error's `userInfo`
5. Custom password rules: set `ASAccountAuthenticationModificationPasswordGenerationRequirements` in the extension's Info.plist

### Sign in with Apple Upgrade Flow (No Step-Up UI)
1. System calls `convertAccountToSignInWithAppleWithoutUserInteraction(for:existingCredential:userInfo:)`
2. Extension authenticates with backend
3. On success: call `extensionContext.getSignInWithAppleUpgradeAuthorization(state:nonce:completionHandler:)` **[NEW]**; system shows system Sign in with Apple sheet
4. After authorization: commit upgrade on server, call `extensionContext.completeUpgradeToSignInWithApple()`
5. After completion: the keychain password credential is deleted by the system
6. On failure or user cancel: cancel request with appropriate error

### Security Step-Up UI (When Additional Auth Is Required)
- When backend requires further user verification, cancel the initial request with `ASAccountAuthenticationModificationErrorCode.userInteractionRequired`
- System creates a new request configured for UI display
- For strong password: calls `prepareInterfaceToChangePassword(for:newPassword:existingCredential:userInfo:)` on the view controller
- For Sign in with Apple: calls `prepareInterfaceToConvertAccountToSignInWithApple(for:existingCredential:userInfo:)`
- System presents the view controller with a nav bar (app name as title) and system Cancel button
- To handle system cancel: override `cancelRequest()`, call `super` (which calls `cancelRequest(withError:)`)

### In-App Upgrades
- Create an `ASAccountAuthenticationModificationStrongPasswordUpgradeRequest` or `ASAccountAuthenticationModificationSignInWithAppleUpgradeRequest` **[NEW]**
- Set `userInfo` to pass auth tokens or other data to the extension for authorization (the app has no password to pass)
- Create `ASAccountAuthenticationModificationController(requestHandler:)`, set its delegate and `presentationContextProvider`
- Call `perform(request:)`
- Delegate methods: `accountAuthenticationModificationController(_:didSuccessfullyComplete:userInfo:)` and `accountAuthenticationModificationController(_:didFail:error:)`
- After successful Sign in with Apple upgrade: system updates or removes the keychain credential for the service

### Domain Association Requirement
- App must associate with the web credential service before upgrades will be offered
- Serve `/.well-known/apple-app-site-association` JSON on the domain listing the app under `webcredentials`
- Add `webcredentials:<domain>` to the app's Associated Domains entitlement in Xcode

## APIs & Frameworks

- **AuthenticationServices**
  - `ASAccountAuthenticationModificationViewController` **[NEW]** — base class for the extension; `UIViewController` subclass
  - `ASAccountAuthenticationModificationViewController.extensionContext: ASAccountAuthenticationModificationExtensionContext` **[NEW]**
  - `ASAccountAuthenticationModificationExtensionContext.completeChangePasswordRequest(updatedCredential:userInfo:)` **[NEW]** — completes strong password upgrade
  - `ASAccountAuthenticationModificationExtensionContext.completeUpgradeToSignInWithApple()` **[NEW]** — completes Sign in with Apple upgrade
  - `ASAccountAuthenticationModificationExtensionContext.getSignInWithAppleUpgradeAuthorization(state:nonce:completionHandler:)` **[NEW]** — requests Sign in with Apple credential for upgrade
  - `ASAccountAuthenticationModificationExtensionContext.cancelRequest(withError:)` **[NEW]** — cancels the upgrade request
  - `ASAccountAuthenticationModificationErrorCode` **[NEW]** — `.failed`, `.userInteractionRequired`, `.userCanceled`
  - `ASExtensionLocalizedFailureReasonErrorKey` **[NEW]** — `userInfo` key for localized failure message
  - `ASAccountAuthenticationModificationController` **[NEW]** — triggers in-app upgrade requests
  - `ASAccountAuthenticationModificationStrongPasswordUpgradeRequest` **[NEW]** — in-app strong password upgrade request; has `serviceIdentifier`, `username`, `userInfo`
  - `ASAccountAuthenticationModificationSignInWithAppleUpgradeRequest` **[NEW]** — in-app Sign in with Apple upgrade request
  - `ASAccountAuthenticationModificationControllerDelegate` **[NEW]** — two methods: success and failure callbacks
  - `ASAccountAuthenticationModificationControllerPresentationContextProviding` **[NEW]** — provides window for UI
  - `ASCredentialServiceIdentifier` — identifies the service domain/URL for the upgrade
  - `ASPasswordCredential` — wraps username + password; used in `completeChangePasswordRequest`
- **Info.plist keys (extension)**
  - `ASAccountAuthenticationModificationSupportStrongPasswordChange` **[NEW]** — `YES`/`NO`
  - `ASAccountAuthenticationModificationSupportsUpgradeToSignInWithApple` **[NEW]** — `YES`/`NO`
  - `ASAccountAuthenticationModificationPasswordGenerationRequirements` **[NEW]** — password rules string

## Code Highlights

Strong password upgrade without step-up UI:
```swift
override func changePasswordWithoutUserInteraction(
    for serviceIdentifier: ASCredentialServiceIdentifier,
    existingCredential: ASPasswordCredential,
    newPassword: String,
    userInfo: [AnyHashable: Any]?) {

    // Authenticate with backend using existing cookies/tokens
    MyBackendClient.shared.changePassword(
        username: existingCredential.user,
        newPassword: newPassword) { success, error in
        if success {
            let updatedCredential = ASPasswordCredential(
                user: existingCredential.user, password: newPassword)
            self.extensionContext.completeChangePasswordRequest(
                updatedCredential: updatedCredential, userInfo: nil)
        } else {
            self.extensionContext.cancelRequest(withError: /* error */)
        }
    }
}
```

In-app strong password upgrade:
```swift
let serviceID = ASCredentialServiceIdentifier(identifier: "shinyapp.example.com",
                                               type: .domain)
let request = ASAccountAuthenticationModificationStrongPasswordUpgradeRequest(
    serviceIdentifier: serviceID,
    username: currentUser.username,
    userInfo: ["authToken": currentUser.token])

let controller = ASAccountAuthenticationModificationController()
controller.delegate = self
controller.presentationContextProvider = self
controller.perform(request)
```

## Takeaways
- The Account Authentication Modification Extension runs out-of-process; it authorizes upgrades by talking to the app's backend, leveraging shared App Group cookies/tokens rather than receiving a plain-text password from the app.
- Both upgrade types (strong password and Sign in with Apple) can be supported in a single extension target; each is declared independently in Info.plist.
- Show step-up UI only when absolutely necessary — cancel with `.userInteractionRequired` to trigger it; keep upgrade flows fast and frictionless.
- After a successful Sign in with Apple upgrade, do not allow the account to log in with a password any longer; the system deletes the keychain credential automatically.
- The `userInfo` property of in-app upgrade requests is the mechanism for passing auth context (tokens, session IDs) to the extension, since the app must not supply a plain-text password.

---
_Source: WWDC20 Session 10666 page (abstract and transcript)._
