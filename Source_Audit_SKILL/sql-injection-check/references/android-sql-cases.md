# Android SQL Injection Source Cases

## Purpose

This file contains Android-specific source point patterns and candidate search terms for SQL injection source discovery.

Use it when Android code constructs or executes local SQL, passes SQL-like selectors into Android storage APIs, or forwards mobile-controlled query options to backend data APIs, especially in:
- SQLiteDatabase and SQLiteOpenHelper code
- Room DAOs using raw queries or dynamic query builders
- ContentProvider query, update, delete, and call paths
- exported components, deep links, WebView bridges, Binder/AIDL handlers, SDK callbacks, push handlers, and background jobs
- local search, filter, sort, sync, cache, offline storage, analytics, and debug/admin database features

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify exported reachability, input control, propagation, downstream SQL relevance, and later binding or structural controls.

---

# 1. High-Coverage Android SQL Source Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; keep it only when code evidence shows that the value can influence SQLite/Room/ContentProvider query values, selection strings, projections, sort order, raw query text, stored query metadata, or backend SQL query options.

## 1.1 Manifest, component, and mobile entry candidates

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
- `android:pathPrefix`
- `android:pathPattern`
- `android:authorities`
- `android:grantUriPermissions`
- `android:readPermission`
- `android:writePermission`
- `android:permission`
- `ACTION_SEARCH`
- `SearchManager.QUERY`

## 1.2 Activity, service, receiver, provider, and callback source APIs

Search for:
- `onCreate`
- `onNewIntent`
- `getIntent`
- `getData`
- `getExtras`
- `getStringExtra`
- `getIntExtra`
- `getParcelableExtra`
- `Intent.getAction`
- `Uri.getQueryParameter`
- `Uri.getQueryParameterNames`
- `Uri.getPathSegments`
- `Uri.getLastPathSegment`
- `onStartCommand`
- `onBind`
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
- `RemoteMessage.getData`
- `Worker.doWork`
- `Data.getString`
- SDK callback payloads

## 1.3 WebView, bridge, and IPC source APIs

Search for:
- `WebView`
- `addJavascriptInterface`
- `@JavascriptInterface`
- `evaluateJavascript`
- `shouldOverrideUrlLoading`
- `shouldInterceptRequest`
- `WebMessagePort`
- `postMessage`
- `onTransact`
- `Binder`
- `IBinder`
- `IInterface`
- `AIDL`
- `Messenger`
- `Handler`
- `Message`
- `Bundle`
- `ResultReceiver`
- `ContentResolver.call`

## 1.4 SQL value and criteria source candidates

Search for Intent, Uri, Bundle, WebView bridge, provider, or worker fields named:
- `q`
- `query`
- `keyword`
- `search`
- `term`
- `name`
- `username`
- `email`
- `status`
- `state`
- `type`
- `category`
- `tenantId`
- `accountId`
- `userId`
- `ids`
- `from`
- `to`
- `startDate`
- `endDate`
- `filters`
- `criteria`
- `conditions`
- `where`
- `selection`
- `selectionArgs`

## 1.5 Projection, selection, sort, and structural source candidates

Search for:
- `projection`
- `columns`
- `column`
- `sortOrder`
- `orderBy`
- `sort`
- `direction`
- `selection`
- `whereClause`
- `having`
- `groupBy`
- `limit`
- `offset`
- `table`
- `tableName`
- `operator`
- `rawQuery`
- `sql`
- `querySql`
- `filter`
- `filterBy`
- `SearchRecentSuggestions`

## 1.6 Raw query and local database relevance candidates

After finding a source candidate, trace toward:
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
- Room `@RawQuery`
- `SupportSQLiteQuery`
- `SimpleSQLiteQuery`
- `RoomDatabase.query`
- `RoomDatabase.compileStatement`

## 1.7 Stored and second-order Android SQL sources

Search for:
- synced remote records later queried locally
- cached filters or saved searches
- recent search suggestions
- local preference values used as sort/filter fields
- downloaded dashboard/report configuration
- push payloads stored and later queried
- import rows saved into SQLite/Room
- content provider writes later used by provider queries
- WebView bridge inputs persisted before local query use

## 1.8 Android graph search recipes

Useful combinations:

```text
android:exported="true" + getStringExtra/getData + selection/sortOrder/projection
ContentProvider.query + Uri.getQueryParameter + selection/sortOrder
@JavascriptInterface + query/filter/orderBy + rawQuery/SimpleSQLiteQuery
ACTION_SEARCH/SearchManager.QUERY + SQLiteDatabase.query/rawQuery
Worker.doWork/RemoteMessage.getData + saved filter + Room @RawQuery
getPathSegments/getLastPathSegment + table/column/id + SQLiteQueryBuilder
ContentResolver.query + projection/sortOrder from Intent/Uri/Bundle
```

---

# 2. Android Source Patterns

## A-S1. Exported component supplies local query criteria

Example idea:
- an exported activity, service, receiver, provider, or deep link reads query, filter, or sort fields and passes them toward local database access.

Audit relevance:
The source is client-controlled or IPC-controlled when another app, browser, WebView, or deep link can supply it.

Follow-up:
- verify caller restrictions, provider permissions, selection argument binding, and structural allowlists.

## A-S2. ContentProvider selection, projection, or sort source

Example idea:
- `query(...)`, `update(...)`, or `delete(...)` accepts `selection`, `selectionArgs`, `projection`, or `sortOrder` and passes them to SQLite APIs.

Audit relevance:
Provider APIs are query surfaces; structural values should be fixed or allowlisted, and values should use selection arguments.

Follow-up:
- inspect provider export status, permissions, strict projection maps, and query builder strict modes.

## A-S3. Room raw query source

Example idea:
- Intent, Bundle, worker input, or stored configuration supplies text used in `SimpleSQLiteQuery` or `@RawQuery`.

Audit relevance:
Room typed DAO methods are usually safer, but raw query sources require binding and structural constraints.

Follow-up:
- verify `SimpleSQLiteQuery` bind args and fixed query templates.

## A-S4. WebView bridge or SDK callback query source

Example idea:
- JavaScript bridge, SDK callback, or push payload provides search/filter/sort values that influence local SQLite or backend query options.

Audit relevance:
These are weakly trusted source points unless caller origin and payload integrity are proven.

Follow-up:
- validate WebView origin checks, bridge exposure, and downstream query handling.

---

# 3. False-Positive Controls

Do not mark an Android source as high-priority if:
- the component is not reachable by external apps or weakly trusted code,
- provider calls require strong permissions and caller identity is checked,
- values are fixed server-side or mapped to strict projection/sort allowlists,
- values are used only as `selectionArgs` with no structural influence,
- Room typed DAO methods receive only bound values and no raw SQL text.

Use `Suspected source` or `Not enough evidence` if:
- export status, caller identity, or provider permissions are unclear,
- Room DAO implementation or `SupportSQLiteQuery` construction is hidden,
- stored writer paths are missing,
- local SQL options are forwarded to backend services and backend behavior is hidden.

---

# 4. Quick Android Source Checklist

- Are exported components, deep links, providers, WebView bridges, or IPC handlers feeding local database queries?
- Do Intent, Uri, Bundle, message, or worker fields influence `selection`, `projection`, `sortOrder`, raw SQL, or Room raw queries?
- Are ContentProvider query/update/delete sources guarded by permissions and strict projection/sort mappings?
- Are saved filters, synced records, recent searches, or push payloads reused in local SQL construction?
- Is the source only a bound value, or can it change table, column, operator, order, projection, limit, or raw query text?
