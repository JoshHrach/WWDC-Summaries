# Meet CKTool JS
**WWDC22 · Session 10116** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10116/)

_Platforms:_ Node.js, web browsers (JavaScript/TypeScript); manages iCloud containers for iOS, iPadOS, macOS, tvOS, watchOS apps

## Overview
CKTool JS is a new JavaScript/TypeScript npm library introduced at WWDC22 that lets developers manage and automate iCloud CloudKit containers from JavaScript. It offers the same operations as the `cktool` command-line utility shipped with Xcode 13 and is the library used internally by CloudKit Console itself. CKTool JS fills a gap that previously required macOS-only tools or manual CloudKit Console interactions: developers can now automate schema resets, schema imports/exports, and record CRUD operations from any Node.js or browser-based JavaScript environment, making it directly integrable into CI/CD pipelines and build tooling.

The library ships with strict TypeScript type definitions, enabling compile-time type checking and IDE code completion. It is distributed as a set of scoped `@apple` npm packages with transparent version history.

## Key Topics
- **Package structure** — the main package is `@apple/cktool.database`; platform packages `@apple/cktool.target.nodejs` and `@apple/cktool.target.browser` provide platform-specific implementations; transitive dependencies `@apple/cktool.core`, `@apple/cktool.api.base`, `@apple/cktool.api.database` are pulled in automatically
- **Authentication tokens** — two types, both obtainable from CloudKit Console: *Management Token* (scoped to team + user; required for schema operations) and *User Token* (scoped to team + container; required for accessing private user data records)
- **Configuration pattern** — create a `security` object with tokens, a `defaultArgs` object with `teamId`/`containerId`/`environment`, call the platform-specific `createConfiguration()`, then instantiate `PromisesApi` with configuration + security
- **Environments** — `CKEnvironment.DEVELOPMENT` for safe schema iteration; `CKEnvironment.PRODUCTION` for live user data
- **Schema management** — CloudKit Schema Language files use `.ckdb` extension; `api.resetToProduction(defaultArgs)` resets dev schema to production state (deletes all types if not yet in production); `api.importSchema({ ...defaultArgs, file })` uploads a schema; `api.exportSchema(defaultArgs)` downloads it
- **Record CRUD** — `api.queryRecords` (filter-based queries), `api.createRecord`, `api.updateRecord` (requires `recordChangeTag`), `api.deleteRecord`; all return promises
- **Type-safe field values** — field values are constructed via `makeRecordFieldValue.*` factory functions (`.int64()`, `.double()`, `.string()`, `.reference()`, etc.); large numbers use `toInt64()` / `toDouble()` coercion helpers; invalid values throw exceptions before hitting the server
- **CloudKit schema concepts** — containers, record types, fields, system fields (`recordName`), references (`CKDBRecordReferenceAction`), zones (`_defaultZone`), database types (`CKDatabaseType.PRIVATE`)

## APIs & Frameworks
**CKTool JS npm packages** **[NEW]**
- `@apple/cktool.database` **[NEW]** — main package; exports `PromisesApi`, `CKEnvironment`, `CKDatabaseType`, `makeRecordFieldValue`, `CKDBQueryFilterType`, `CKDBRecordReferenceAction`, `toInt64`, `toDouble`
- `@apple/cktool.target.nodejs` **[NEW]** — Node.js platform package; exports `createConfiguration`, `File`
- `@apple/cktool.target.browser` **[NEW]** — browser platform package; exports `createConfiguration`

**PromisesApi methods** **[NEW]**
- `api.resetToProduction(defaultArgs)` — resets development schema to production state
- `api.exportSchema(defaultArgs)` — exports the container's current schema
- `api.importSchema({ ...defaultArgs, file: File })` — imports a `.ckdb` schema file into the container
- `api.queryRecords({ ...databaseArgs, body: { query: { recordType, filters } } })` — queries records; returns `response.result.records`
- `api.createRecord({ ...databaseArgs, body: { recordType, fields } })` — creates a new record; returns `response.result.record`
- `api.updateRecord({ ...databaseArgs, recordName, body: { recordType, recordChangeTag, fields } })` — updates an existing record (requires `recordChangeTag`); returns `response.result.record`
- `api.deleteRecord({ ...databaseArgs, recordName })` — deletes a record by name

**Field value factory functions** (from `makeRecordFieldValue`) **[NEW]**
- `makeRecordFieldValue.int64(value)` — creates an Int64 field value
- `makeRecordFieldValue.double(value)` — creates a Double field value
- `makeRecordFieldValue.string(value)` — creates a String field value
- `makeRecordFieldValue.reference({ recordName, action: CKDBRecordReferenceAction })` — creates a record reference

**Enumerations** **[NEW]**
- `CKEnvironment.DEVELOPMENT` / `CKEnvironment.PRODUCTION`
- `CKDatabaseType.PRIVATE` / `CKDatabaseType.PUBLIC` / `CKDatabaseType.SHARED`
- `CKDBQueryFilterType.EQUALS` (and other filter types)
- `CKDBRecordReferenceAction.DELETE_SELF` / `CKDBRecordReferenceAction.NONE`

## Code Highlights
Configure CKTool JS for Node.js:
```javascript
const { CKEnvironment, PromisesApi } = require("@apple/cktool.database");
const { createConfiguration } = require("@apple/cktool.target.nodejs");

const security = {
    "ManagementTokenAuth": "<YOUR_MANAGEMENT_TOKEN>",
    "UserTokenAuth": "<YOUR_USER_TOKEN>"
};
const defaultArgs = {
    "teamId": "<YOUR_TEAM_ID>",
    "containerId": "<YOUR_CONTAINER_ID>",
    "environment": CKEnvironment.DEVELOPMENT
};
const configuration = createConfiguration();
const api = new PromisesApi({ "configuration": configuration, "security": security });
```

Reset to production then import a schema:
```javascript
const { File } = require("@apple/cktool.target.nodejs");
const fs = require("fs/promises");

const importMySchema = async () => {
    const schemaPath = "MyApp.ckdb";
    const buffer = await fs.readFile(schemaPath);
    const file = new File([buffer], schemaPath);
    await api.importSchema({ ...defaultArgs, "file": file });
};

api.resetToProduction(defaultArgs).then(() => importMySchema());
```

Query records with a filter:
```javascript
const { CKDBQueryFilterType, makeRecordFieldValue } = require("@apple/cktool.database");

const response = await api.queryRecords({
    ...databaseArgs,
    "body": {
        "query": {
            "recordType": "Countries",
            "filters": [{
                "fieldName": "isoCode3",
                "fieldValue": makeRecordFieldValue.string("USA"),
                "type": CKDBQueryFilterType.EQUALS
            }]
        }
    }
});
const countryRecord = response.result.records[0];
```

Create a record with typed field values:
```javascript
const { makeRecordFieldValue, CKDBRecordReferenceAction } = require("@apple/cktool.database");

const response = await api.createRecord({
    ...databaseArgs,
    "body": {
        "recordType": "Coins",
        "fields": {
            "country": makeRecordFieldValue.reference({
                recordName: countryRecord.recordName,
                action: CKDBRecordReferenceAction.DELETE_SELF
            }),
            "issueYear": makeRecordFieldValue.int64(2007),
            "nominalValue": makeRecordFieldValue.double(0.10)
        }
    }
});
```

## Takeaways
- CKTool JS brings CloudKit container management — schema resets, schema import/export, and full record CRUD — to any JavaScript/TypeScript environment, enabling automation in CI pipelines without requiring macOS or Xcode.
- Strict TypeScript types and client-side value validation (via factory functions) catch errors before requests reach the server, reducing wasted round-trips.
- Always `resetToProduction` before importing a new development schema to start from a clean baseline; chain promises to enforce ordering.
- `updateRecord` requires the `recordChangeTag` from the current record — always fetch the record first to get its latest tag before updating.

---
_Source: WWDC22 Session 10116 page (abstract, chapter summaries, code samples, and resource links)._
