# Meet CloudKit Console
**WWDC21 · Session 10117** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10117/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
CloudKit Console is a redesigned web-based dashboard at developer.apple.com that replaces the old CloudKit Dashboard. It provides a unified, intuitive interface for managing CloudKit containers across all phases of the development lifecycle: schema design, data exploration, schema deployment, and production monitoring.

The Console is organized into three apps — Database, Telemetry, and Logs — each offering purpose-built tools. The consistent layout places container and account context at the top, navigation on the left, and detail content in the center, following familiar Apple application conventions. Deep-linkable URLs allow bookmarking specific views directly in Safari.

## Key Topics

**Database App**
The core of CloudKit Console. Covers schema management and data querying for both development and production environments. Supports just-in-time schema creation during early development and formal schema definition through the Console UI.

**Schema Management**
Record types are listed with their fields and indexes. Adding fields requires only a name and type selection. Indexes can be viewed and created per field with three index type options. Security roles for the public database now have a dedicated new UI.

**Deploy Schema Changes**
A diff view shows exactly what changes will be applied when promoting from development to production. Because production schema changes are irreversible, this view gives developers a clear confirmation step before deployment. CloudKit verifies integrity to avoid breaking older clients.

**Query Builder**
A new left-to-right contextual query builder lets developers select database, zone, and record type, then apply filters to retrieve specific records. Queries can be saved for repeated use during development.

**Telemetry App**
Charts showing key CloudKit usage metrics: request rate, server latency, error count, and average request size — all filterable by time range and other dimensions. Helps identify when app behavior changes after a release.

**Logs App**
Detailed logging output of how CloudKit has processed each app request. Useful for debugging both in development and production.

## APIs & Frameworks

### CloudKit Console (Web Tool — no client-side API changes)
- **CloudKit Console** — redesigned dashboard at `developer.apple.com` **[NEW]**
- **Database App** — schema management and record querying
  - Record types — define fields (String, Int64, Double, Bytes, Date/Time, Location, Reference, Asset, List) and index types (Queryable, Sortable, Searchable)
  - Security roles — manage public database access controls
  - "Deploy Schema Changes" — diff view for dev-to-production promotion **[NEW UI]**
  - Query builder — filter-based record retrieval with saveable queries **[NEW]**
- **Telemetry App** — charts for request rate, server latency, error count, avg request size **[NEW]**
- **Logs App** — detailed CloudKit server-side request logs **[NEW]**

### CloudKit (Framework — referenced context)
- `CKContainer` — CloudKit container scoping data
- `CKDatabase` — development vs. production environments
- `CKRecord` / `CKRecordType` — data structure mapped to record types in Console
- `CKRecordZone` — database zone context for queries
- Security roles (public database) — `CKRecord.RecordType` access control

## Code Highlights

No client-side code samples in this session. CloudKit Console is a web tool at developer.apple.com.

## Takeaways
- CloudKit Console replaces the old CloudKit Dashboard with a redesigned three-app structure (Database, Telemetry, Logs) at `developer.apple.com/icloud/cloudkit/`.
- The new Deploy Schema Changes diff view provides a clear, safe confirmation step before irreversible production promotions.
- The saveable Query Builder and deep-linkable Console URLs substantially reduce repetitive overhead during active CloudKit development.
- The Telemetry and Logs apps give developers post-ship visibility into how their apps are actually using CloudKit, enabling data-driven debugging and optimization.

---
_Source: WWDC21 Session 10117 page (abstract, chapter summaries, code samples, and resource links)._
