# Optimizing Storage in Your App
**WWDC19 · Session 419** · [Watch](https://developer.apple.com/videos/play/wwdc2019/419/)

_Platforms:_ iOS 13, macOS Catalina 10.15, watchOS 6, tvOS 13

## Overview
Efficient storage is critical to app performance, battery life, and overall device health. This session covers a wide range of optimization techniques including selecting the right image format, understanding file system metadata costs, controlling how data is synced to disk, and choosing the right data persistence mechanism.

Core Data and SQLite each receive dedicated attention, with practical guidance on journaling modes, transaction strategies, vacuuming, and indexing. The session also introduces updates to the File Activity instrument in Instruments 11, which helps developers identify anti-patterns and verify storage improvements.

Attendees leave with concrete strategies for reducing on-disk footprint, minimizing unnecessary I/O, and making smarter choices about file creation and deletion.

## Key Topics

### Efficient Image Assets
- HEIC (High Efficiency Image Container) offers 50% smaller files at comparable quality to JPEG, and supports alpha, lossless, depth data, and multi-image containers. iOS 11+ and macOS High Sierra+ support it natively.
- Asset Catalogs store image assets in a single optimized format, enable App Store slicing (users download only device-appropriate assets), and support GPU-based compression with hardware-accelerated decompression.
- Adopting HEIC + Asset Catalogs reduced a demo app from 24.6 MB to 14.9 MB — a 40% reduction.

### File System Metadata Costs
- Every file creation, deletion, or rename triggers APFS copy-on-write metadata operations. Creating one small file results in ~12 KB of I/O for just 240 bytes of data (~2% efficiency).
- Avoid rapidly creating and deleting temporary files. Instead, create files, keep them open and unlinked, and do not call fsync — this lets the OS cache them efficiently.

### Syncing to Disk
- `fsync`: moves data from OS cache to disk cache, but not necessarily to permanent storage. May be redundant if the OS does it periodically.
- `fcntl(F_FULLFSYNC)`: flushes the entire disk cache to permanent storage — expensive and often unnecessary.
- `fcntl(F_BARRIERFSYNC)`: **[NEW in iOS 13]** enforces I/O ordering without flushing the full disk cache. Preferred alternative to `F_FULLFSYNC` when ordering guarantees are needed.

### Serialized Data Files (plist/XML/JSON)
- Any single change requires re-serializing and rewriting the entire file. Poor scalability and high file-system metadata overhead.
- Best for infrequently modified, small data. Not a replacement for a database.

### Core Data
- Built on SQLite; handles object graphs, change tracking, conflict resolution, connection pooling, schema migration, memory management, and statement aggregation automatically.
- **[NEW]** CloudKit integration in iOS 13.
- **[NEW]** Data denormalization support in iOS 13.
- Adopters typically write 50–70% less model-layer code.

### SQLite Best Practices
- **Connections:** Keep the database open as long as possible; pool connections in multi-threaded apps.
- **Journaling:** WAL (Write Ahead Log) mode is far more efficient than default Delete mode — merges writes to the same page, uses fewer barriers, supports concurrent readers.
- **Transactions:** Batch multiple INSERT/UPDATE/DELETE statements into single transactions so SQLite can merge page writes.
- **File Size/Privacy:** Use `secure_delete=fast` (default in iOS 13) to zero deleted data efficiently. Use `auto_vacuum=INCREMENTAL` instead of `VACUUM` for compaction.
- **Indexes:** Use partial indexes (with a WHERE clause) to limit indexing overhead to rows that actually benefit.

### File Activity Instrument (Instruments 11)
- Now supports all Apple devices (iOS, macOS, watchOS, tvOS).
- Shows logical I/O vs. physical I/O together.
- **[NEW]** Automated anti-pattern detection: excessive physical writes, failed I/O system calls, suboptimal cache usage.
- Tracks: File System Suggestions, File System Activity (logical reads/writes), Disk Usage, Disk I/O Latency.

## APIs & Frameworks

### Image & Asset APIs
- `HEIC` / `HEIF` image format **[NEW widespread adoption encouraged]**
- Asset Catalogs (`.xcassets`) — app slicing, GPU-based compression
- On-Demand Resources

### File System & Sync
- `fsync()` — flush to disk cache
- `fcntl(F_FULLFSYNC)` — flush disk cache to permanent storage
- `fcntl(F_BARRIERFSYNC)` — I/O barrier ordering hint **[NEW]**
- APFS copy-on-write semantics
- `NSDictionary.write(toFile:atomically:)` — triggers fsync internally

### Core Data
- `NSPersistentContainer` — Core Data stack setup
- `NSManagedObjectContext` — context for object graph operations
- `NSFetchedResultsController` — live query support
- `NSPersistentCloudKitContainer` — CloudKit integration **[NEW in iOS 13]**
- `NSBatchInsertRequest` / `NSBatchDeleteRequest` — bulk operations
- Automatic schema migrations
- Data denormalization **[NEW in iOS 13]**

### SQLite Pragmas & Modes
- `PRAGMA journal_mode=WAL` — Write Ahead Log journaling mode
- `PRAGMA secure_delete=fast` — efficient secure deletion (default in iOS 13) **[NEW default]**
- `PRAGMA auto_vacuum=INCREMENTAL` — incremental vacuuming
- `PRAGMA incremental_vacuum(N)` — vacuum N pages at a time
- Partial indexes: `CREATE INDEX … WHERE <condition>`
- `sqlite3_open()` / `sqlite3_close()` — connection management

### Instruments
- File Activity instrument (Instruments 11) **[UPDATED]**
- File System Suggestions track **[NEW]**
- Disk Usage track
- Disk I/O Latency track
- File System Activity track (logical I/O)

## Code Highlights

WAL mode journaling pragma:
```sql
PRAGMA journal_mode=WAL;
```

Secure delete (for older iOS versions):
```sql
PRAGMA secure_delete=fast;
```

Partial index example:
```sql
CREATE INDEX idx_favorites ON photos (id) WHERE is_favorite = 1;
```

Multiple statements in a single transaction:
```sql
BEGIN TRANSACTION;
DELETE FROM photos WHERE id = 1;
DELETE FROM photos WHERE id = 2;
DELETE FROM photos WHERE id = 3;
COMMIT;
```

Incremental vacuum:
```sql
PRAGMA auto_vacuum=INCREMENTAL;
PRAGMA incremental_vacuum(10);
```

## Takeaways
- Replace JPEG with HEIC and use Asset Catalogs to cut app size by up to 40% with minimal effort.
- Prefer `F_BARRIERFSYNC` over `F_FULLFSYNC` for I/O ordering — it's significantly cheaper.
- Always use WAL journaling mode for SQLite; batch statements into single transactions to reduce physical I/O dramatically.
- Use the updated File Activity instrument to detect and verify I/O optimizations — the automated suggestions track makes anti-patterns visible at a glance.

---
_Source: WWDC19 Session 419 page (abstract, chapter summaries, code samples, and resource links)._
