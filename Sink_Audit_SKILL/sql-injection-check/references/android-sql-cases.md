# Android SQL Injection Cases

## Purpose

This file contains Android-specific SQL injection patterns, candidate sink inventories, and audit cases.

Use it when the target application includes Android code that constructs or executes local SQL, especially in:
- SQLiteDatabase and SQLiteOpenHelper code
- Room DAOs using raw queries or dynamic query builders
- ContentProvider query/update/delete paths
- exported components, deep links, WebView bridges, IPC handlers, SDK callbacks, and background jobs that pass selectors into local database APIs
- search, filter, sort, sync, cache, analytics, offline storage, and admin/debug local database features

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify exported reachability, attacker-controlled input, query construction behavior, sink execution, and missing binding or structural controls.

---

# 1. Android SQL Control Points

## 1.1 Entry and IPC points
Look for:
- exported components
- content providers
- intent filters and deep links
- WebView bridge methods
- Binder/AIDL/Messenger calls
- push notification callbacks
- WorkManager and background services
- sync/import paths that update local SQL state

## 1.2 Query construction and execution points
Look for:
- SQLite raw query APIs
- Room raw query APIs
- ContentProvider selection strings
- dynamic projection, sortOrder, groupBy, having, and limit values
- direct string-built SQL for local search or sync

## 1.3 Safety controls
Look for:
- `selectionArgs`
- bound SQLite statements
- Room typed DAO methods
- fixed projection and sort mappings
- caller identity checks for providers
- strict allowlists for structural elements

---

# 2. High-Coverage Android SQL Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm reachability, input control, sink behavior, and missing controls.

## 2.1 Manifest and component entry candidates
Search for:
- `android:exported="true"`
- `android:exported`
- `<activity`
- `<service`
- `<receiver`
- `<provider`
- `<intent-filter>`
- `<action`
- `<data`
- `android:scheme`
- `android:host`
- `android:path`
- `android:permission`
- `android:readPermission`
- `android:writePermission`
- `android:authorities`
- `android:grantUriPermissions`

## 2.2 Activity, service, receiver, provider, and callback entries
Search for:
- `onCreate`
- `onNewIntent`
- `getIntent`
- `getData`
- `getExtras`
- `onStartCommand`
- `onBind`
- `Worker.doWork`
- `BroadcastReceiver`
- `onReceive`
- `ContentProvider`
- `query`
- `insert`
- `update`
- `delete`
- `bulkInsert`
- `applyBatch`
- `call`
- `FirebaseMessagingService`
- `onMessageReceived`
- `ACTION_SEARCH`
- `SearchManager.QUERY`

## 2.3 WebView, bridge, and IPC entries
Search for:
- `WebView`
- `addJavascriptInterface`
- `@JavascriptInterface`
- `evaluateJavascript`
- `shouldOverrideUrlLoading`
- `shouldInterceptRequest`
- `onTransact`
- `Binder`
- `IBinder`
- `IInterface`
- `AIDL`
- `Messenger`
- `Handler`
- `Message`
- `ResultReceiver`

## 2.4 SQLite, Room, and ContentProvider sink candidates
Search for:
- `SQLiteDatabase.rawQuery`
- `rawQuery`
- `execSQL`
- `SQLiteDatabase.query`
- `SQLiteQueryBuilder`
- `SQLiteStatement`
- `compileStatement`
- `SQLiteOpenHelper`
- `getReadableDatabase`
- `getWritableDatabase`
- `ContentResolver.query`
- `ContentResolver.update`
- `ContentResolver.delete`
- `ContentProvider.query`
- `ContentProvider.update`
- `ContentProvider.delete`
- `selection`
- `selectionArgs`
- `sortOrder`
- `projection`
- `groupBy`
- `having`
- `limit`
- `@RawQuery`
- `SupportSQLiteQuery`
- `SimpleSQLiteQuery`
- `RoomDatabase.query`
- `RoomDatabase.compileStatement`
- `@Query`

## 2.5 Query construction and structural fragment candidates
Search for:
- `String sql`
- `String query`
- `String selection`
- `String sortOrder`
- `String whereClause`
- `StringBuilder`
- `String.format`
- `+ " WHERE " +`
- `+ " ORDER BY " +`
- `+ " LIMIT " +`
- `getQueryParameter`
- `getStringExtra`
- `Uri.getLastPathSegment`
- `Uri.getPathSegments`
- `Base64.decode`
- `URLDecoder.decode`
- `filter`
- `sort`
- `column`
- `table`
- `operator`
- `projection`

## 2.6 Required-control candidates
Search near sinks for:
- `selectionArgs`
- `?` placeholders
- `bindString`
- `bindLong`
- `bindDouble`
- `SQLiteStatement.bind`
- Room typed DAO methods
- no `@RawQuery`
- `SimpleSQLiteQuery` with bind args
- `SQLiteQueryBuilder.setProjectionMap`
- `SQLiteQueryBuilder.setStrict`
- `SQLiteQueryBuilder.setStrictColumns`
- `SQLiteQueryBuilder.setStrictGrammar`
- projection allowlist
- sort allowlist
- `android:exported="false"`
- `android:permission`
- `checkCallingPermission`
- `getCallingUid`
- `validate`
- `allowlist`

## 2.7 Android graph search recipes
Useful combinations:

```text
android:exported="true" + ContentProvider.query
ContentProvider.query + selection
Uri.getQueryParameter + rawQuery
getStringExtra + execSQL
@JavascriptInterface + rawQuery
ACTION_SEARCH + SQLiteDatabase.query
sortOrder + request/Intent value
@RawQuery + SimpleSQLiteQuery
SimpleSQLiteQuery + string concatenation
SQLiteQueryBuilder + missing projection map
```

---

# 3. Android SQL Injection Anti-Patterns

### A1. ContentProvider raw selection concatenation
```java
String selection = "name = '" + uri.getQueryParameter("q") + "'";
return db.query("users", null, selection, null, null, null, sortOrder);
```

Why risky:
URI-controlled input changes SQL selection logic.

### A2. rawQuery from Intent extra
```java
String sql = "select * from item where name like '%" + getIntent().getStringExtra("q") + "%'";
db.rawQuery(sql, null);
```

Why risky:
Intent input is inserted into executable SQL text.

### A3. Room RawQuery with dynamic SQL
```java
new SimpleSQLiteQuery("select * from logs where " + whereClause);
```

Why risky:
Raw Room queries bypass typed DAO protections when fragments are attacker-controlled.

### A4. Dynamic sortOrder from caller
```java
return db.query("items", projection, selection, args, null, null, sortOrder);
```

Why risky:
`sortOrder` is a structural SQL element and requires allowlisting.

---

# 4. Case Templates

## Case A-SQL-1: Exported provider SQL injection

### Vulnerable pattern
```java
db.rawQuery("select * from users where name = '" + q + "'", null);
```

### Audit focus
Verify exported provider reachability, input control, query construction, and use of bind arguments.

## Case A-SQL-2: Room raw query structural injection

### Vulnerable pattern
```java
SimpleSQLiteQuery query = new SimpleSQLiteQuery("select * from item order by " + sort);
```

### Audit focus
Verify whether `sort` is fixed through a strict mapping.

## Case A-SQL-3: ContentResolver caller controls sortOrder

### Vulnerable pattern
```java
resolver.query(uri, projection, selection, args, sortOrder);
```

### Audit focus
Verify whether caller-controlled sort or projection reaches provider SQL without allowlisting.

---

# 5. Android-Specific Audit Heuristics

## 5.1 ContentProvider heuristics
Review `query`, `update`, `delete`, `bulkInsert`, and `applyBatch` methods for selection, projection, and sort controls.

## 5.2 SQLite heuristics
Treat `rawQuery`, `execSQL`, and `compileStatement` as high-priority sinks when SQL text is dynamic.

## 5.3 Room heuristics
Room typed `@Query` methods are usually safer for values, but `@RawQuery`, `SupportSQLiteQuery`, and `SimpleSQLiteQuery` require full review.

## 5.4 Exported reachability heuristics
Prioritize SQL paths reachable from exported providers, deep links, bridge methods, Binder calls, and push/SDK callbacks.
