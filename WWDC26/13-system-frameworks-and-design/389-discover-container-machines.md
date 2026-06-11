# Discover container machines
**WWDC26 · Session 389** · [Watch](https://developer.apple.com/videos/play/wwdc2026/389/)

_Platforms:_ macOS

## Overview
Container machines is a new feature shipped as part of Apple's open-source `container` command-line tool. It provides a lightweight, persistent Linux environment on Mac — sitting above ephemeral OCI containers but below a full VM — and is designed as the most ergonomic way for Mac developers to run, test, and iterate on Linux code without leaving their macOS workflow.

The feature is built on the open-source `Containerization` Swift framework (introduced at WWDC25), which provides VM-based isolation, OCI image support, and sub-second start times. Container machines add stateful persistence across sessions, automatic mirroring of the current macOS user and working directory into the Linux environment, and first-class integration with the `container` CLI.

The demo shows building and testing a Vapor web server from macOS — editing in Xcode, running `swift run` inside a container machine, and previewing in Safari — all without manually copying files or setting up SSH.

## Key Topics

### Containerization Framework
The underlying open-source Swift framework from WWDC25. Each container runs in its own lightweight VM for strong isolation, starts in under a second, and supports standard OCI images. The companion `container` CLI tool is the primary user interface.

### Design Principles (four pillars)
1. **Fast and lightweight** — sub-second start via the Containerization VM layer; minimal overhead
2. **Simple to create and operate** — single CLI command to create and run; no Dockerfile or daemon required
3. **Persistent across sessions** — unlike ephemeral containers, a container machine retains its state between `run` invocations
4. **Seamless macOS extension** — automatic user mirroring (same UID/username inside Linux), automatic working directory mirroring, so `pwd` and file paths work identically on both sides

### Container Machine Architecture
A container machine is backed by a persistent OCI-based disk image. On each `container machine run`, the Containerization framework boots the VM, mounts the macOS user's home directory, and drops into a shell (or runs a command) as the mirrored user. Because the disk is persistent, installed packages and state survive across runs.

### CLI Demo Walkthrough
- `container machine` — lists available subcommands
- `container machine create --name demo --set-default alpine` — create a machine from the Alpine image
- `container machine run echo hi` — run a single command non-interactively
- `container machine run uname` — verify the Linux kernel
- `container machine run` — open an interactive shell
- `container machine list` — show all machines and their states
- `swift run` — run a Swift Package target inside the container machine

## APIs & Frameworks

### Containerization (open-source Swift framework on GitHub)
- `container` CLI tool — the primary user-facing interface; includes the new `machine` subcommand **[NEW]**
- `container machine create [--name <name>] [--set-default] <image>` **[NEW]** — creates a persistent machine from an OCI image
- `container machine run [<command>]` **[NEW]** — executes a command (or opens an interactive shell) inside the default machine
- `container machine list` **[NEW]** — shows all machines and their running state
- OCI image support — machines are backed by standard OCI-compatible images (e.g., Alpine, Ubuntu)
- Automatic user mirroring **[NEW]** — UID and username from macOS propagated into the Linux environment
- Automatic directory mirroring **[NEW]** — macOS working directory is accessible at the same path inside the machine

### Related Open-Source Repositories
- `github.com/apple/container` — the CLI tool; container 1.0 release
- `github.com/apple/containerization` — the underlying Swift framework

## Code Highlights

This session is primarily a CLI/tooling session. The representative commands are:

```sh
# Create a container machine from Alpine, set as default
container machine create --name demo --set-default alpine

# Run a single command
container machine run echo hi
container machine run uname

# Open an interactive shell
container machine run

# List all machines
container machine list

# Build and run a Swift package from inside the machine
swift run
```

No Swift API code samples were shown (the Containerization framework Swift API was covered in the WWDC25 "Meet Containerization" session).

## Takeaways
- Container machines give Mac developers a persistent, fast Linux environment with zero file-copying friction — the macOS working directory and user identity are automatically mirrored inside.
- The feature is part of the open-source `container` 1.0 release on GitHub; no Xcode or App Store install needed.
- Built on the Containerization framework's VM isolation model, so each machine is secure and independent despite feeling like a local shell.
- Pairs naturally with Xcode for editing + container machine for building/testing Linux targets, avoiding the need for a remote Linux server.

---
_Source: WWDC26 Session 389 page (abstract, chapter summaries, code samples, and resource links)._
