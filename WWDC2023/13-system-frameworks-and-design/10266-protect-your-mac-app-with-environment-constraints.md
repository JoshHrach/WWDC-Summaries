# Protect your Mac app with environment constraints
**WWDC23 · Session 10266** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10266/)

_Platforms:_ macOS Sonoma 14, macOS Ventura 13.3+

## Overview
Mac apps frequently consist of multiple processes — helper tools, XPC services, launch agents, launch daemons, frameworks, and libraries — any of which could be exploited to escalate privileges if they can be launched by arbitrary processes or loaded with untrusted code. This session introduces **environment constraints**, a new macOS security feature that lets developers declaratively describe the conditions under which their processes are allowed to run and the code allowed to load into their address space.

Environment constraints were first used internally in macOS Ventura to protect OS processes; in macOS Sonoma they are available for third-party developers. They complement existing protections (App Sandbox, Hardened Runtime, Library Validation, Gatekeeper/Notarization) by adding a new dimension: not just *what* the code is, but *how* and *from where* it is allowed to run. Constraints are encoded as property list dictionaries embedded into code signatures and evaluated by the kernel at launch and library-load time.

## Key Topics

### Threat Model
Parent processes have broad control over child processes via `posix_spawn` — they can inject environment variables, Mach ports, and loaded libraries. Malicious processes that can write to disk can tamper with plists, replace binaries, or remove runtime protections. Environment constraints address these threats at the process-launch level.

### Launch Constraints
Embedded into a binary's code signature; prevent the binary from running unless specified conditions about itself, its parent process, or its responsible process are satisfied. Three flavors:
- **Self constraint** — properties the process itself must have (e.g., must be launched as an application via Launch Services)
- **Parent process constraint** — properties the process's direct parent must have (e.g., only `MyDemo.app` may spawn the helper)
- **Responsible process constraint** — properties the process responsible for the launch must have (e.g., only `MyDemo.app` may be responsible for the XPC service)

When a launch constraint is violated, the launch is blocked and a crash report is generated.

### Launchd Plist Constraints (`SpawnConstraint`)
A `SpawnConstraint` key in a launchd plist (for launch agents/daemons registered via `SMAppService`) restricts which binary the plist is allowed to launch. Prevents attackers who gain persistent execution via user-approved background items from replacing the target binary.

### Library Load Constraints
Embedded into a process's code signature; describe which code may be loaded into the process's address space. More fine-grained than the existing `disable-library-validation` entitlement — allows loading specific third-party libraries (by team identifier, signing identifier, or cdhash) while blocking all others. Apple-signed code cannot be excluded.

### Constraint Dictionary Structure
Constraints are plist dictionaries with keys representing facts and operators:
- **Facts:** `signing-identifier`, `team-identifier`, `cdhash`
- **Operators:** `$and`, `$or`, `$and-array`, `$or-array`, `$in`
- Top-level keys are implicitly ANDed

### Availability
- Launch constraints: enforced from macOS 13.3+
- Library load constraints and launchd plist constraints: enforced from macOS Sonoma 14+
- All constraint types are optional; adoption does not break backward compatibility

## APIs & Frameworks

- **Security framework** — underlying framework for code signing and constraints
- **Code signing** — constraints are embedded at signing time, not at runtime
- `signing-identifier` — constraint fact: unique identifier for a binary (from `codesign` output, `identifier` field) **[NEW]**
- `team-identifier` — constraint fact: Apple Developer team ID **[NEW]**
- `cdhash` — constraint fact: unique hash of a specific binary version **[NEW]**
- `$and` — constraint operator: all predicates in the dictionary must be true **[NEW]**
- `$or` — constraint operator: at least one predicate in the dictionary must be true **[NEW]**
- `$and-array` — constraint operator: array of `[$op, dict]` pairs, all must be true **[NEW]**
- `$or-array` — constraint operator: array of `[$op, dict]` pairs, at least one must be true **[NEW]**
- `$in` — constraint operator: fact value must be in the specified array **[NEW]**
- **Xcode signing configuration settings:**
  - `Launch Constraint Parent Process Plist` — path to parent process constraint plist **[NEW]**
  - `Launch Constraint Responsible Process Plist` — path to responsible process constraint plist **[NEW]**
  - `Library Load Constraint Plist` — path to library load constraint plist **[NEW]**
- `SpawnConstraint` key in launchd plist — constrains which binary the plist may launch **[NEW]**
- **`SMAppService`** — API for registering launch agents/daemons; constraint enforcement applies to registered plists
- **Hardened Runtime** — existing protection; library load constraints provide a more targeted alternative to `disable-library-validation`
- **Library Validation** — existing entitlement; allows only team-signed or Apple-signed code; constraints are more flexible
- **App Sandbox** — existing protection; complementary to environment constraints
- **Gatekeeper / Notarization** — existing protection; complementary to environment constraints
- `codesign` command-line tool — inspect existing constraints and signatures

## Code Highlights

Parent process constraint plist (only `MyDemo.app` may launch the helper):
```xml
<dict>
    <key>team-identifier</key>
    <string>M2657GZ2M9</string>
    <key>signing-identifier</key>
    <string>com.demo.MyDemo</string>
</dict>
```

Responsible process constraint plist (multiple processes in the bundle allowed):
```xml
<dict>
    <key>team-identifier</key>
    <string>M2657GZ2M9</string>
    <key>signing-identifier</key>
    <dict>
        <key>$in</key>
        <array>
            <string>com.demo.MyDemo</string>
            <string>com.demo.DemoMenuBar</string>
            <string>demohelper</string>
        </array>
    </dict>
</dict>
```

Launchd plist with `SpawnConstraint` (prevents binary replacement attacks):
```xml
<key>SpawnConstraint</key>
<dict>
    <key>team-identifier</key>
    <string>M2657GZ2M9</string>
    <key>signing-identifier</key>
    <string>com.demo.DemoMenuBar</string>
</dict>
```

Library load constraint allowing code from two teams:
```xml
<dict>
    <key>team-identifier</key>
    <dict>
        <key>$in</key>
        <array>
            <string>M2657GZ2M9</string>
            <string>P9Z4AN7VHQ</string>
        </array>
    </dict>
</dict>
```

## Takeaways

- Environment constraints are optional but provide a new layer of defense for multi-process Mac apps, particularly those with helper tools, XPC services, or third-party libraries that carry elevated privileges.
- Use **parent process launch constraints** to ensure only your main app can spawn privileged helpers; use **responsible process constraints** for XPC services that should only be reachable from your app bundle.
- Add a `SpawnConstraint` to launchd plists registered via `SMAppService` to prevent attackers from hijacking user-approved background execution by replacing your binary.
- Use **library load constraints** as a precise alternative to the blanket `disable-library-validation` entitlement when you need to load a specific third-party library.

---
_Source: WWDC23 Session 10266 page (abstract, chapter summaries, code samples, and resource links)._
