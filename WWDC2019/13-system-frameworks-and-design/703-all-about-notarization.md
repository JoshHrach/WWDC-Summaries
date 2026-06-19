# All About Notarization
**WWDC19 · Session 703** · [Watch](https://developer.apple.com/videos/play/wwdc2019/703/)

_Platforms:_ macOS Catalina 10.15 (Developer ID distribution)

## Overview
Notarization is an automated security vetting service for Mac software distributed outside the Mac App Store, introduced at WWDC18 and now required for all new Developer ID-signed software (signed on or after June 1, 2019) to pass Gatekeeper on macOS Catalina. The session covers the end-to-end notarization workflow, correct code signing requirements (inside-out signing, code places, secure timestamps), the Hardened Runtime and its entitlements, and the command-line tools used to submit, check, staple, and verify notarized software.

99% of submissions receive a response within 15 minutes. The Notary Service status is tracked on Apple's public status page. Anyone on the developer team can submit (role restriction removed since WWDC18).

## Key Topics

### What Notarization Provides
- Identifies and blocks malicious software prior to distribution without requiring App Review or the Mac App Store.
- Extension of the Developer ID program — same certificates, developer stays in control of signing and distribution.
- The Notary Service performs automated security checks (not human review).
- Benefits: prevents inadvertent shipping of malicious dependencies; hardened runtime makes apps more secure by default; provides user confidence and audit trail of all submissions from a Developer ID account.

### New Requirement: Notarization Required in macOS Catalina **[NEW]**
- All software signed with a Developer ID certificate **on or after June 1, 2019** must be notarized to pass Gatekeeper.
- Previously distributed software with only a Developer ID signature continues to pass Gatekeeper unchanged — the requirement is only for newly signed builds.
- At runtime (Gatekeeper verification): checks the stapled local ticket and also contacts the Notary Service via CloudKit for a revocation check.

### Correct Code Signing Requirements

**Sign everything:**
- All Mach-O binaries, all bundles, installer packages, and disk images.
- Sign with Developer ID Application certificate + secure timestamp (`--timestamp` flag).
- Installer packages: sign with Developer ID Installer certificate (different from Application certificate).
- Disk images: sign with Developer ID Application certificate + `--timestamp` to avoid Gatekeeper path randomization.
- Executables must opt into the Hardened Runtime; dylibs, frameworks, and bundles do not need it.

**Inside-out signing:**
- Sign the most deeply nested bundle or binary first, then work outward to the top-level bundle.
- The `--deep` flag is insufficient — it only finds code in code places, misses dylibs and bundles in non-standard locations. Always prefer explicit inside-out signing for custom workflows.
- Reference Technote 2206 for code places and inside-out signing details.

**Code places:**
- Files in code places within a bundle (e.g., `Contents/MacOS`, `Contents/Frameworks`, `Contents/PlugIns`, `Contents/XPCServices`, `Contents/Library/LoginItems`) must have code signatures.
- Mach-Os and bundles embed signatures; other file types (JPEG, raw binary) get signatures as extended attributes — be careful these extended attributes survive packaging (use `ditto` or Archive Utility).
- Recommendation: put non-Mach-O files outside code places to avoid extended-attribute complications.

**Never invalidate a signature:**
- Do not modify files inside a bundle except during installation or update.
- After an update, the result must be correctly signed and notarized on the customer's system.
- If modifying files at runtime is necessary for the updater: always create a new file and move the old one out of the way — do not overwrite existing files (code signatures are latched in the kernel on first use; overwriting a running file causes a code-signature violation).

### Hardened Runtime **[NEW in macOS Mojave, required for Notarization in Catalina]**
Enable in Xcode: Signing & Capabilities → "Hardened Runtime" capability. Command-line: `codesign --sign <identity> --timestamp --options runtime`.
Verify: `codesign -dvvv <binary>` — look for `runtime` in the flags line.

**Runtime Code Signing Enforcement (on by default):**
- Verifies all memory mapped from disk matches its code signature (including non-executable sections).
- Prevents execution of modified memory that doesn't match its signature.
- JIT exception: use `com.apple.security.cs.allow-jit` entitlement + `MAP_JIT` flag when allocating JIT memory.
- If `MAP_JIT` is unavailable (no JIT source access): `com.apple.security.cs.allow-unsigned-executable-memory` — weakens to allowing unsigned executable regions.
- Patching system frameworks: `com.apple.security.cs.allow-unsigned-executable-memory` — not recommended.

**Library Validation (on by default):**
- Only code signed by the same Developer ID team or Apple can be loaded into the process.
- Prevents code injection and dylib hijacking.
- Plug-in ecosystems: prefer out-of-process plug-in model; if in-process is required: `com.apple.security.cs.disable-library-validation` entitlement.

**DYLD Environment Variable Protection (on by default):**
- Blocks DYLD_INSERT_LIBRARIES and similar environment variables at runtime.
- For debugging only: `com.apple.security.get-task-allow` — Xcode adds/removes automatically for debug/release builds. Do not include in Notary Service submissions (rejected).
- If shipping with DYLD variables: `com.apple.security.cs.allow-dyld-environment-variables` (accepted by Notary Service; not recommended).

**Debugging Protection (on by default):**
- Prevents debugger attachment to hardened processes.
- `com.apple.security.get-task-allow` entitlement enables debugging (along with DYLD variables).
- Caution: debugger attachment disables Runtime Code Signing Enforcement; always test a release build (without get-task-allow) to catch RCSE-related issues.
- Plug-in debugging: Notary Service accepts `get-task-allow` + `disable-library-validation` together for developer debug builds.

**Protected Resource Entitlements:**
- Camera, microphone, location, contacts, calendar, etc. — declare `NSUsageDescription` string + corresponding entitlement on the main bundle.
- Entitlements on main bundle only; do not propagate to all executables/helpers.

### Notarization Workflow

**Submission formats accepted:** `.dmg` (disk image), `.pkg` (installer package), `.zip` (zip archive with macOS metadata preserved via `ditto` or Archive Utility). Do not submit `.app` directly — wrap in one of these formats.

**Custom installers:** if the installer downloads content at install time or uses custom packaging, do 2-step notarization: notarize the final disk content separately, then notarize the installer app separately.

**Tools:**

- `xcrun altool --notarize-app --primary-bundle-id <id> --username <AppleID> --password @keychain:<item> --file <path>` — submit for notarization; returns `RequestUUID`.
- `xcrun altool --notarization-info <RequestUUID> --username <AppleID> --password ...` — poll status; returns status, log file URL (valid ~24 hours), issues array.
- `xcrun stapler staple <path>` — staple ticket to `.app`, `.dmg`, or `.pkg` (not `.zip` directly — unzip, staple, rezip).
- `xcrun stapler validate <path>` — verify stapling.
- `spctl --assess --verbose <app>` — verify notarization of an app bundle; look for `source=Notarized Developer ID`.
- `spctl --assess --verbose --type install <pkg>` — verify installer package.
- `spctl --assess --verbose --type open --context context:primary-signature <dmg>` — verify signed disk image.
- `codesign --verify --verbose --test-requirement="=notarized" <binary>` — verify notarization of any binary/file; look for `explicit requirement satisfied`.
- `xcrun altool --notarization-history 0 --username <AppleID> --password ...` — list submission history for the account.

**Xcode workflow:** Archive → Organizer → Distribute App → Developer ID → Upload → export notarized app (Xcode staples automatically).

## APIs & Frameworks

**Command-Line Tools (Xcode 10+)**
- `xcrun altool --notarize-app` — submit for notarization **[NEW workflow; submission role expanded]**
- `xcrun altool --notarization-info` — query submission status
- `xcrun altool --notarization-history` — audit trail of account submissions
- `xcrun stapler staple` — attach ticket to software
- `xcrun stapler validate` — verify ticket attachment
- `spctl --assess` — Gatekeeper assessment (existing; `Notarized Developer ID` source value added)
- `codesign --options runtime` — adopt Hardened Runtime
- `codesign --display -vvv` — verify Hardened Runtime adoption (check for `runtime` in flags)

**Hardened Runtime Entitlements**
- `com.apple.security.cs.allow-jit` — allow JIT compilation with `MAP_JIT`
- `com.apple.security.cs.allow-unsigned-executable-memory` — allow unsigned executable memory regions
- `com.apple.security.cs.disable-library-validation` — allow loading unsigned/adhoc-signed plug-ins
- `com.apple.security.cs.allow-dyld-environment-variables` — allow DYLD environment variables at runtime
- `com.apple.security.get-task-allow` — allow debugger attachment and DYLD vars (development only; usually rejected by Notary Service)
- Resource entitlements: `com.apple.security.device.camera`, `com.apple.security.device.microphone`, `com.apple.security.personal-information.location`, etc.

## Code Highlights

```bash
# Inside-out signing (custom workflow)
# 1. Sign most deeply nested content first
codesign --sign "Developer ID Application: My Company" \
         --timestamp \
         "Watching Grass Grow.app/Contents/Frameworks/Sparkle.framework/Versions/A/Updater.app"

codesign --sign "Developer ID Application: My Company" \
         --timestamp \
         "Watching Grass Grow.app/Contents/Frameworks/Sparkle.framework"

codesign --sign "Developer ID Application: My Company" \
         --timestamp \
         "Watching Grass Grow.app/Contents/Library/LoginItems/Watching Grass Grow Helper.app"

codesign --sign "Developer ID Application: My Company" \
         --timestamp \
         "Watching Grass Grow.app/Contents/MacOS/savegrowgrass.dylib"

# 2. Sign top-level bundle last
codesign --sign "Developer ID Application: My Company" \
         --timestamp \
         --options runtime \
         "Watching Grass Grow.app"

# Submit for notarization
xcrun altool --notarize-app \
             --primary-bundle-id "com.example.watchgrassgrow" \
             --username "dev@example.com" \
             --password "@keychain:AltoolPassword" \
             --file "WatchingGrassGrow.dmg"
# → Success: RequestUUID = <uuid>

# Poll status
xcrun altool --notarization-info <uuid> \
             --username "dev@example.com" \
             --password "@keychain:AltoolPassword"
# → Status: success, logFileUrl: https://...

# Staple
xcrun stapler staple "Watching Grass Grow.app"
xcrun stapler validate "Watching Grass Grow.app"

# Verify Gatekeeper acceptance
spctl --assess --verbose "Watching Grass Grow.app"
# → source=Notarized Developer ID

# Verify Hardened Runtime adoption
codesign -dvvv "Watching Grass Grow.app/Contents/MacOS/WatchingGrassGrow"
# → flags=0x10000(runtime)  ← must be present
```

## Takeaways
- All new Developer ID software (signed on or after June 1, 2019) must be notarized to pass Gatekeeper on macOS Catalina — Developer ID certificate alone is no longer sufficient.
- Use inside-out signing in custom workflows; `codesign --deep` is not sufficient and leaves non-code-place binaries unsigned.
- Take only the Hardened Runtime entitlements you actually need — each one reduces a security protection; entitlements are publicly inspectable in the signed binary.
- Staple tickets to all distributed software so Gatekeeper can verify offline without contacting the Notary Service.
- The `altool --notarization-history` command provides a full audit trail — useful for discovering unauthorized releases from your Developer ID account.

---
_Source: WWDC19 Session 703 page (transcript, abstract, and resource links)._
