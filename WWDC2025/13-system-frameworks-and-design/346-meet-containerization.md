# Meet Containerization
**WWDC25 · Session 346** · [Watch](https://developer.apple.com/videos/play/wwdc2025/346/)

_Platforms:_ macOS

## Overview
Apple open-sourced a full Linux container runtime for macOS, written entirely in Swift. The Containerization framework and its companion `container` CLI tool provide a native, Virtualization.framework-backed way to pull, run, and manage OCI-compatible Linux container images directly on Apple Silicon Macs — without Docker Desktop or any third-party daemon.

The stack is built from first principles in Swift: a lightweight `vminitd` init system runs inside the VM, the EXT4 Swift package handles the Linux filesystem layer, and the Swift Static Linux SDK produces the init binary as a fully statically linked executable. The result is a fast, resource-efficient container runtime that starts VMs in well under a second.

The session explains why Apple built a new runtime rather than packaging an existing one, walks through the open-source architecture, and demonstrates the `container` CLI with a live interactive session.

## Key Topics

### Architecture Overview
Each container runs as an isolated lightweight VM via Apple's Virtualization.framework. Unlike Docker on macOS (which runs a single Linux VM and shares it across all containers), each Containerization container gets its own VM, providing strong isolation with minimal overhead on Apple Silicon.

### Containerization Swift Package
The public Swift package exposes the runtime API for programmatic container lifecycle management: pulling images from OCI registries, creating and starting containers, mounting volumes, and inspecting running processes. Third-party tools (CI systems, IDEs, dev environment managers) can embed this package.

### EXT4 Swift Package
A pure-Swift EXT4 filesystem implementation used to construct and mount the Linux root filesystem image inside the VM. This package is also independently useful for tools that need to read or write Linux disk images on macOS.

### vminitd — Swift Init System
The PID 1 inside the VM is `vminitd`, a minimal init system written in Swift and compiled with the Swift Static Linux SDK. It manages process spawning, signal forwarding, and clean VM shutdown. Because it uses the Static Linux SDK, it carries no dynamic library dependencies — the binary is self-contained.

### `container` CLI
```
container image pull ubuntu:24.04
container run -t -i ubuntu:24.04 /bin/bash
```
The CLI supports standard OCI image operations (pull, list, remove) and container operations (run, stop, exec, logs). Interactive TTY sessions work out of the box.

## APIs & Frameworks

- **Containerization Swift package** **[NEW, open source]** — programmatic OCI container lifecycle API for macOS
- **`container` CLI tool** **[NEW, open source]** — command-line interface for container management
- **EXT4 Swift package** **[NEW, open source]** — pure-Swift Linux EXT4 filesystem read/write
- **vminitd** **[NEW, open source]** — Swift-based PID 1 init system for VM containers
- **Swift Static Linux SDK** — produces fully statically linked Linux binaries from Swift on macOS
- **Virtualization.framework** — underlying VM host (Apple private framework, used internally)
- **OCI image format** — standard container image specification (Docker Hub, GHCR, etc. compatible)

## Code Highlights

```bash
# Pull and run an interactive Ubuntu container
container image pull ubuntu:24.04
container run -t -i ubuntu:24.04 /bin/bash
```

```swift
// Programmatic usage via the Containerization Swift package
import Containerization

let registry = OCIRegistry(host: "registry-1.docker.io")
let image = try await registry.pull(reference: "ubuntu:24.04")
let container = try Container(image: image)
try await container.start()
try await container.exec(["/bin/bash"])
```

## Takeaways

- The Containerization stack is fully open source and Swift-native — contribute, embed, or fork it without licensing concerns.
- Per-container VM isolation is stronger than shared-VM runtimes; each container cannot see another's filesystem or processes by default.
- The EXT4 package is independently valuable for any macOS tool that needs to construct or inspect Linux disk images.
- Containerization targets developer workflows on Apple Silicon; it is not a production server runtime, but it is an excellent local dev and CI environment tool.

---
_Source: WWDC25 Session 346 page (abstract, chapter summaries, code samples, and resource links)._
