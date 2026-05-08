# Android File Path Handling Source Cases

## Purpose

This file contains Android-specific source point patterns and candidate search terms for path traversal and unsafe file path handling source discovery.

Use it when Android code can introduce path-like values, especially:
- exported activities, services, receivers, and content providers
- deep links and app links
- WebView JavaScript bridges
- Binder, AIDL, Messenger, PendingIntent, and IPC handlers
- Storage Access Framework, document providers, MediaStore, and content URI flows
- WorkManager jobs, SDK callbacks, file import/export, backup/restore, and cache cleanup logic

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. High-Coverage Android Source Candidate Inventory

## 1.1 Manifest, component, and mobile entry candidates

- `android:exported="true"`
- `<activity>`
- `<service>`
- `<receiver>`
- `<provider>`
- `<intent-filter>`
- deep links
- app links
- `onCreate`
- `onNewIntent`
- `onStartCommand`
- `onReceive`
- `ContentProvider.query`
- `ContentProvider.insert`
- `ContentProvider.update`
- `ContentProvider.delete`
- `openFile`
- `openAssetFile`
- `openTypedAssetFile`
- `WorkManager`
- `Worker`
- `CoroutineWorker`
- `FirebaseMessagingService.onMessageReceived`
- `PendingIntent`
- `ActivityResultLauncher`
- `onActivityResult`
- `@JavascriptInterface`
- `addJavascriptInterface`
- Binder/AIDL methods
- SDK callback interfaces

## 1.2 Intent, URI, Bundle, and IPC path source candidates

- `getIntent()`
- `Intent.getData()`
- `Uri.getPath()`
- `Uri.getEncodedPath()`
- `Uri.getQueryParameter(...)`
- `getStringExtra(...)`
- `getParcelableExtra(...)`
- `getSerializableExtra(...)`
- `getExtras()`
- `Bundle.getString(...)`
- `Bundle.getParcelable(...)`
- `Parcel`
- `readString`
- `readParcelable`
- `AIDL`
- `Messenger`
- `ContentValues`
- `Cursor`
- `DocumentsContract.getDocumentId`
- `MediaStore`
- `OpenableColumns.DISPLAY_NAME`
- `ClipData`
- `ACTION_OPEN_DOCUMENT`
- `ACTION_CREATE_DOCUMENT`
- `ACTION_GET_CONTENT`

## 1.3 Path-like field and selector candidates

- `file`
- `filename`
- `fileName`
- `path`
- `filePath`
- `dir`
- `directory`
- `folder`
- `uri`
- `contentUri`
- `documentUri`
- `documentId`
- `displayName`
- `mediaPath`
- `resource`
- `asset`
- `template`
- `cacheKey`
- `download`
- `export`
- `import`
- `destination`
- `target`
- `backup`
- `restore`
- `cleanupTarget`

## 1.4 Path construction and storage candidates

- `File`
- `Paths.get`
- `Path`
- `Uri.parse`
- `getFilesDir`
- `getCacheDir`
- `getExternalFilesDir`
- `getExternalStorageDirectory`
- `Environment`
- `canonicalPath`
- `canonicalFile`
- `absolutePath`
- `normalize`
- `resolve`
- `FileProvider`
- `ContentResolver.openInputStream`
- `ContentResolver.openOutputStream`
- `DocumentFile.fromSingleUri`
- `DocumentFile.fromTreeUri`
- `AssetManager.open`
- `Resources.openRawResource`
- `openFileInput`
- `openFileOutput`

## 1.5 Archive, stored, and second-order candidates

- `ZipInputStream`
- `ZipFile`
- `ZipEntry.getName`
- `JarEntry.getName`
- imported package entries
- backup manifests
- Room database path fields
- SQLite path fields
- SharedPreferences stored paths
- offline queue paths
- WorkManager input data
- push payload paths
- WebView local storage
- downloaded config paths
- SDK cached file metadata

## 1.6 Android graph search recipes

```text
android:exported/deep link + Uri/getStringExtra path/fileName/contentUri + File/ContentResolver
@JavascriptInterface + file/path/export/download + openFile/ContentResolver/backend file API
ContentProvider + query/openFile + Uri path/documentId/displayName
ACTION_OPEN_DOCUMENT/ACTION_GET_CONTENT + displayName/documentId + copy/openInputStream
ZipEntry + getName + File/Path/output stream
WorkManager/FirebaseMessagingService + inputData/payload path + delete/export/copy
```

---

# 2. Android Source Patterns and Blind Spots

## A-S1. Exported component path values

Intent extras, deep-link parameters, and content URIs can become source points when another app or external link can influence them.

## A-S2. Content URI and document ID sources

`content://` URIs, `DocumentsContract` IDs, and MediaStore display names are source points when converted to paths, filenames, or local copies.

## A-S3. WebView bridge file arguments

Bridge arguments that carry file names, export names, storage keys, or path fragments deserve tracing into local file APIs and backend file operations.

## A-S4. Stored mobile file paths

Room/SQLite records, SharedPreferences, offline queues, WorkManager input data, and cached SDK metadata can become second-order path sources.

---

# 3. False-Positive Controls

Do not mark an Android source as high-priority if:
- the component is not externally reachable and no indirect trigger is visible,
- the value is only a content URI handled by provider APIs without conversion to unsafe filesystem paths,
- the filename is replaced by a server/app-generated safe name before use,
- the downstream operation is display-only and not file/resource handling.

Use `Suspected source` or `Not enough evidence` if:
- component reachability is unclear,
- SAF/provider behavior is hidden,
- WebView bridge origin restrictions are hidden,
- stored path writer paths are missing.

---

# 4. Quick Android Source Checklist

- Are exported components receiving file names, paths, content URIs, or document IDs?
- Are WebView bridge methods receiving file, export, download, or storage key arguments?
- Are content provider or SAF values converted into local paths or filenames?
- Are uploaded/downloaded/cache names stored and later reused?
- Are archive entries extracted from attacker-supplied files?
- Are cleanup/delete/export paths sourced from Intent, IPC, queue, or storage values?
