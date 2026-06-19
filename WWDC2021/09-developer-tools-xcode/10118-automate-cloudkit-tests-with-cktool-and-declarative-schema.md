# Automate CloudKit Tests with cktool and Declarative Schema
**WWDC21 · Session 10118** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10118/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
This session introduces two new tools that dramatically simplify CloudKit integration testing: `cktool`, a command-line utility installed with Xcode 13 that communicates directly with the CloudKit server, and the CloudKit Schema Language, a new text-based declarative format (`.ckdb` files) for defining CloudKit record types and indexes.

Together, these tools solve a persistent integration testing challenge: keeping the CloudKit container schema and seed data in sync with the app's data model before each test run. By placing the schema file in source control and running `cktool` as an Xcode test pre-action script, teams can guarantee a consistent container state every time tests execute.

## Key Topics

### The Integration Testing Problem
- CloudKit integration tests need the server schema to exactly match the app's data model
- Tests that modify cloud data leave the container in an inconsistent state for subsequent runs
- Previously, both problems required manual CloudKit Console work

### cktool Command-Line Tool
- **New** tool bundled with Xcode 13 **[NEW]**; invoked via `xcrun cktool`
- Communicates directly with CloudKit server without requiring a running app
- Operations are synchronous — suitable for scripting (failure in one step stops the script)
- Key commands:
  - `save-token` — store management or user token in macOS Keychain
  - `get-teams` — list developer team memberships
  - `export-schema` — download current development schema to a `.ckdb` file
  - `import-schema` — upload a `.ckdb` schema file to the container
  - `reset-schema` — reset the development schema
  - `create-record` — create a record with JSON-encoded fields
  - `query-records` — query existing records

### Authorization: Management and User Tokens
- **Management token** **[NEW]**: scoped to container configuration (schema import/export); not for data access; tied to developer account, works across teams; obtained from CloudKit Console
- **User token** **[NEW]**: authorizes data access to private or public databases; obtained from CloudKit Console
- Tokens stored securely in macOS Keychain via `xcrun cktool save-token`

### CloudKit Schema Language (`.ckdb` files)
- **New** declarative file format for describing CloudKit schemas **[NEW]**
- Human-readable; supports single-line (`//`) and multi-line comments
- Store in project source tree and check into version control
- Contents: `DEFINE SCHEMA { RECORD TYPE <name> { ... }; }`
- Per-record-type: field names, data types, optional index declarations, security role grants
- Field data types: `STRING`, `INT64`, `TIMESTAMP`, `REFERENCE`, `BYTES`, etc.
- Index types: `QUERYABLE`, `SEARCHABLE`, `SORTABLE` (declared inline after the type)
- Security roles: `_creator` (record creator only), `_icloud` (any authenticated user), `_world` (all users)
- Permissions: `GRANT READ/WRITE/CREATE TO "<role>"`

### Schema Evolution Rules (unchanged in CloudKit)
- Development environment: add/remove record types and custom fields freely
- Production environment: can add new record types and new fields; cannot delete or rename record types or fields once promoted
- Indexes and security roles can be modified in production

### Xcode Test Pre-Action Integration
1. Edit Xcode scheme → Test → Pre-actions → New Run Script Action
2. Provide build settings from app target (so `$PROJECT_DIR` is available)
3. Script: `reset-schema` → `import-schema` from project file → `create-record` for seed data
4. Failures in the script abort test launch, preventing tests from running against invalid data

## APIs & Frameworks

### cktool CLI (xcrun cktool) **[NEW]**
- `xcrun cktool save-token --type management` — store management token
- `xcrun cktool save-token --type user` — store user token
- `xcrun cktool get-teams` — list teams
- `xcrun cktool export-schema --team-id <ID> --container-id <ID> --environment development --output-file schema.ckdb`
- `xcrun cktool import-schema --team-id <ID> --container-id <ID> --environment development --file schema.ckdb`
- `xcrun cktool reset-schema --team-id <ID> --container-id <ID>`
- `xcrun cktool create-record --team-id <ID> --container-id <ID> --environment development --database-type public --record-type <Type> --fields-json '<JSON>'`

### CloudKit Schema Language **[NEW]**
- File extension: `.ckdb`
- Syntax: `DEFINE SCHEMA { RECORD TYPE <Name> ( <field> <TYPE> [<INDEX>], ..., GRANT <PERMISSION> TO "<role>" ); }`

## Code Highlights

Exporting schema:
```bash
xcrun cktool export-schema \
  --team-id XYZ1234567 \
  --container-id iCloud.com.WWDC21.Example \
  --environment development \
  --output-file schema.ckdb
```

Schema language file example:
```
DEFINE SCHEMA
    RECORD TYPE Book (
        "___createTime" TIMESTAMP,
        "___createdBy"  REFERENCE,
        "___etag"       STRING,
        "___modTime"    TIMESTAMP,
        "___modifiedBy" REFERENCE,
        "___recordID"   REFERENCE QUERYABLE,
        description     STRING,
        pageCount       INT64,
        publishedOn     TIMESTAMP,
        // A single-line comment
        title           STRING QUERYABLE,
        GRANT WRITE TO "_creator",
        GRANT CREATE TO "_icloud",
        GRANT READ TO "_world"
    );
```

Test pre-action script:
```bash
xcrun cktool reset-schema \
    --team-id XYZ1234567 \
    --container-id iCloud.com.WWDC21.Example

xcrun cktool import-schema \
    --team-id XYZ1234567 \
    --container-id iCloud.com.WWDC21.Example \
    --environment development \
    --file $PROJECT_DIR/Example/CloudKitSchema.ckdb

xcrun cktool create-record \
    --team-id XYZ1234567 \
    --container-id iCloud.com.WWDC21.Example \
    --environment development \
    --database-type public \
    --record-type Book \
    --fields-json '{"title": {"type": "stringType", "value": "Great Expectations"}, ...}'
```

## Takeaways
- `cktool` and the CloudKit Schema Language together make CloudKit integration testing reproducible and automatable without any GUI interaction.
- Checking the `.ckdb` file into source control ties schema evolution to code changes — the same version of the app always uses the same schema.
- Adding the three-step script (reset → import → seed) as an Xcode test pre-action ensures every test run starts from a known-good container state.
- The management token is separate from user-data access — use the appropriate token type for each operation to minimize credential scope.

---
_Source: WWDC21 Session 10118 page (abstract, chapter summaries, code samples, and resource links)._
