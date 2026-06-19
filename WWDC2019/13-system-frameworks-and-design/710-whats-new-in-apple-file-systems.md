# What's New in Apple File Systems
**WWDC19 · Session 710** · [Watch](https://developer.apple.com/videos/play/wwdc2019/710/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15

## Overview
This session covers three major file system changes shipping in 2019. First, macOS Catalina introduces a Read-Only System Volume — the root filesystem is split into a separate read-only system volume and a writable data volume, joined by a new APFS construct called a Firmlink. Second, APFS volume replication via Apple Software Restore (ASR) is redesigned for Catalina to handle encryption and the non-contiguous nature of APFS volumes, with new snapshot and snapshot-delta restore capabilities for efficient incremental imaging. Third, iOS and iPadOS gain access to external file systems: USB storage (APFS, HFS+, FAT, ExFAT) and SMB 3.0 network shares, accessible through the Files app and any app linked against iOS 13.

## Key Topics

**Read-Only System Volume (macOS Catalina)**
APFS's flexible volume management enables a new security architecture. During the Catalina upgrade, the single main volume is split into a System volume (read-only, holds OS) and a Data volume (writable, holds user content). The two form a Volume Group treated as one entity by the UI — shared encryption, same password, and presented as a single disk.

**Firmlinks**
A new APFS filesystem object that stitches System and Data volumes into a unified directory hierarchy. Unlike symlinks, Firmlinks support bidirectional path traversal (walking up and down the tree returns a consistent path), which is essential for applications that insist on living in specific absolute paths (e.g., `/Applications`). Firmlinks are one-to-one, cannot cross volume group boundaries, and are created by the installer — they are transparent to users and apps.

**Developer Impact of Read-Only Root**
- Apps or installers that write to directories under the root (outside user-writable locations) will break
- Backup utilities relying on inode numbers or filesystem IDs must re-test; assumptions about a single unified volume are no longer valid
- Developers can test by creating `/System/Volumes/Data/ReadOnlySystemVolume` to force read-only mounting in the developer preview
- SIP must be disabled to mount root read-write; this reverts on every reboot

**APFS Volume Replication with ASR**
Block-copy-based replication no longer works for APFS because volumes are non-contiguous and internal storage on T2 Macs uses hardware-bound encryption. ASR now integrates with APFS to generate a streaming copy that handles decryption on the fly (for encrypted sources) and re-encryption on write (for encrypted targets). The resulting stream is also defragmented and metadata-compacted, making ASR a useful mastering step. ASR supports: restoring to an existing volume (target erased), or restoring to a new volume created inside an existing container.

**Snapshot Replication and Delta Restores**
ASR can now restore APFS snapshots to a target volume. More powerfully, it supports snapshot delta restores: if both source and target already share a common snapshot, only the difference (delta) between two snapshots needs to be transferred. This dramatically reduces network bandwidth and time for lab imaging workflows.

**External File Access on iOS/iPadOS (NEW)**
- USB storage: CF cards, USB flash drives, USB RAID boxes; supported filesystems: unencrypted APFS, unencrypted HFS+, FAT, ExFAT
- Compatible with all iOS/iPadOS devices (USB-C natively; Lightning via adapters)
- SMB 3.0 network shares over Wi-Fi, cellular, or Ethernet; Windows Search Protocol (WSP) supported for server-side search including macOS Catalina's built-in SMB server
- Filesystem access in a dedicated process (privilege/process separation for security)
- Available to any app linked on or after iOS 13 — rebuild to get access

**Developer Considerations for External Volumes**
- Case sensitivity: FAT/ExFAT are case-insensitive; HFS+ and APFS can vary — check volume capabilities
- `clonefile()` may not be available on external volumes
- Check `NSURL.resourceValues(forKeys:)` for volume capabilities before using advanced filesystem features
- Keep temp files near working files (same filesystem) to enable atomic save via `rename()`; use `FileManager.url(for: .itemReplacementDirectory, in: .userDomainMask, appropriateFor: documentURL, create: true)` to get a temporary directory on the same volume
- External devices can disconnect unexpectedly; handle SIGBUS from `mmap` on vanished files by using `NSData.ReadingOptions.mappedIfSafe`
- External devices have higher latency than internal APFS — use concurrent I/O operations for sizable transfers

## APIs & Frameworks

**APFS / File System**
- APFS Volume Group **[NEW]** — system + data volume pair treated as a single entity
- Firmlink **[NEW]** — APFS filesystem object for bidirectional traversal between system and data volumes
- Read-Only System Volume **[NEW macOS Catalina]** — `/` mounted read-only; writable user content on Data volume
- `ASR` (Apple Software Restore) APFS integration **[NEW]** — streaming volume replication with on-the-fly encryption handling
- `asr restore --source <vol> --target <vol>` — existing-volume restore
- `asr restore --source <vol> --target <container>` — new-volume restore
- `asr restore --source <snapshot> --target <vol>` **[NEW]** — snapshot restore
- `asr restore --from-snapshot <snap1> --to-snapshot <snap2>` **[NEW]** — snapshot delta restore

**Foundation / File System APIs**
- `NSURL.resourceValues(forKeys:)` with `URLResourceKey` volume capability keys — check filesystem features before using them
  - `URLResourceKey.volumeSupportsCloningKey` — whether `clonefile()` is available
  - `URLResourceKey.volumeSupportsCaseSensitiveNamesKey` — case sensitivity
- `FileManager.url(for:in:appropriateFor:create:)` with `.itemReplacementDirectory` — get temp dir on same volume as target file
- `NSData.ReadingOptions.mappedIfSafe` **[NEW hint]** — prefer memory-mapping only when the backing file is on a reliable (non-removable) volume

**Files App / UIKit**
- Files app external volume support **[NEW]** — USB storage and SMB shares appear as locations
- `UIDocumentPickerViewController` — gains access to external volume locations automatically
- File Provider extensions continue to work for cloud storage

## Code Highlights

Getting a temporary file URL on the same volume as the document (safe atomic save):

```swift
let tempDir = try FileManager.default.url(
    for: .itemReplacementDirectory,
    in: .userDomainMask,
    appropriateFor: documentURL,  // ensures same filesystem
    create: true
)
let tempURL = tempDir.appendingPathComponent(UUID().uuidString)
// write to tempURL, then:
try FileManager.default.replaceItemAt(documentURL, withItemAt: tempURL)
```

Checking volume case sensitivity before a file operation:

```swift
let values = try documentURL.resourceValues(forKeys: [.volumeSupportsCaseSensitiveNamesKey])
let caseSensitive = values.volumeSupportsCaseSensitiveNames ?? true
```

## Takeaways
- macOS Catalina's read-only root is the biggest file system change in years — any installer, tool, or app that writes to system-owned paths must be updated before Catalina ships.
- Firmlinks provide seamless path transparency across the system/data split; apps that use `FileManager` and standard paths generally need no changes.
- ASR snapshot delta restores make large-scale lab imaging dramatically more efficient — only transfer the diff between OS versions rather than full images.
- Any iOS app relinked against iOS 13 automatically gains access to USB storage and SMB network shares through the standard file-picking and file-provider APIs.

---
_Source: WWDC19 Session 710 page (abstract, transcript, and resource links)._
