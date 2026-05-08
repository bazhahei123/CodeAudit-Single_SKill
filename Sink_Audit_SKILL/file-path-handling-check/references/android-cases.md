# Android Path Traversal Cases

## Purpose

This file contains Android-specific path traversal, unsafe local file handling, arbitrary file access, unsafe extraction, content provider path abuse, and WebView local resource loading cases.

Use it when the target application includes Android code, especially in:
- exported activities, services, receivers, and content providers
- deep links and app links
- WebView resource loading and JavaScript bridges
- share targets, file import/export, backup/restore, and attachment workflows
- Binder, AIDL, Messenger, and Intent-based IPC
- ZIP/APK/archive extraction and local cache handling

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify exported reachability, attacker-controlled path input, actual file operation, and missing containment controls.

---

# 1. Android Path Control Points

## 1.1 Entry and IPC points
Look for:
- exported components
- intent filters and deep links
- content provider methods
- WebView bridges and URL interception
- share targets and file-open actions
- Binder/AIDL/Messenger handlers
- push notification handlers
- background work and import/export jobs

## 1.2 File operation points
Look for:
- app-private file reads and writes
- external storage reads and writes
- cache and temp-file handling
- content URI resolution
- asset/resource/template loading
- archive extraction
- backup/restore paths

## 1.3 Path construction and validation
Look for:
- `File` path concatenation
- URI path extraction
- decoded deep-link path segments
- original upload/share filenames
- canonical path checks
- symlink behavior
- scoped storage assumptions

---

# 2. High-Coverage Android Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker reachability, path flow, sink behavior, and missing controls.

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
- `<category`
- `<data`
- `android:scheme`
- `android:host`
- `android:path`
- `android:pathPrefix`
- `android:pathPattern`
- `android:mimeType`
- `android:permission`
- `android:readPermission`
- `android:writePermission`
- `android:grantUriPermissions`
- `android:authorities`
- `android:resource`

## 2.2 Activity, service, receiver, provider, and deep-link entries
Search for:
- `onCreate`
- `onNewIntent`
- `onActivityResult`
- `registerForActivityResult`
- `getIntent`
- `getAction`
- `getData`
- `getDataString`
- `getExtras`
- `onStartCommand`
- `onBind`
- `onHandleIntent`
- `Worker.doWork`
- `BroadcastReceiver`
- `onReceive`
- `ContentProvider`
- `query`
- `insert`
- `update`
- `delete`
- `openFile`
- `openAssetFile`
- `openTypedAssetFile`
- `call`
- `FirebaseMessagingService`
- `onMessageReceived`
- `ACTION_VIEW`
- `ACTION_SEND`
- `ACTION_SEND_MULTIPLE`
- `Intent.EXTRA_STREAM`
- `Intent.EXTRA_TEXT`

## 2.3 WebView, bridge, and IPC entries
Search for:
- `WebView`
- `addJavascriptInterface`
- `@JavascriptInterface`
- `shouldOverrideUrlLoading`
- `shouldInterceptRequest`
- `WebResourceResponse`
- `evaluateJavascript`
- `loadUrl`
- `postMessage`
- `onTransact`
- `Binder`
- `IBinder`
- `IInterface`
- `AIDL`
- `Messenger`
- `Handler`
- `Message`
- `ResultReceiver`
- `FileProvider`
- `getUriForFile`

## 2.4 Path construction and transformation candidates
Search for:
- `new File`
- `File(parent, child)`
- `getFilesDir`
- `getCacheDir`
- `getExternalFilesDir`
- `getExternalCacheDir`
- `Environment.getExternalStorageDirectory`
- `Uri.getPath`
- `Uri.getLastPathSegment`
- `Uri.getPathSegments`
- `Uri.decode`
- `URLDecoder.decode`
- `DocumentsContract.getDocumentId`
- `OpenableColumns.DISPLAY_NAME`
- `getCanonicalPath`
- `getAbsolutePath`
- `normalize`
- `replace("../", "")`
- `substring`
- `split`
- `Base64.decode`

## 2.5 File read, preview, WebView, and content sink candidates
Search for:
- `FileInputStream`
- `openFileInput`
- `contentResolver.openInputStream`
- `ContentResolver.openInputStream`
- `AssetManager.open`
- `Resources.openRawResource`
- `FileReader`
- `BufferedReader`
- `BitmapFactory.decodeFile`
- `BitmapFactory.decodeStream`
- `MediaMetadataRetriever.setDataSource`
- `WebView.loadUrl`
- `WebView.loadDataWithBaseURL`
- `WebResourceResponse`
- `shouldInterceptRequest`
- `openFile`
- `openAssetFile`
- `ParcelFileDescriptor.open`

## 2.6 File write, upload-save, copy, move, delete, and metadata sink candidates
Search for:
- `FileOutputStream`
- `openFileOutput`
- `contentResolver.openOutputStream`
- `ContentResolver.openOutputStream`
- `FileWriter`
- `RandomAccessFile`
- `Files.write`
- `File.delete`
- `deleteFile`
- `File.renameTo`
- `copyTo`
- `FileChannel.transferTo`
- `FileChannel.transferFrom`
- `mkdir`
- `mkdirs`
- `createNewFile`
- `DocumentFile.delete`
- `DocumentFile.createFile`
- `DocumentFile.renameTo`

## 2.7 Archive extraction candidates
Search for:
- `ZipInputStream`
- `ZipFile`
- `ZipEntry`
- `entry.name`
- `entry.getName`
- `getNextEntry`
- `JarFile`
- `GZIPInputStream`
- `TarArchiveInputStream`
- `ArchiveInputStream`
- `APK`
- `extract`
- `unzip`
- `copyInputStreamToFile`

## 2.8 Required-control candidates
Search near sinks for:
- `android:exported="false"`
- `android:permission`
- `checkCallingPermission`
- `checkCallingOrSelfPermission`
- `getCallingPackage`
- `getCallingUid`
- `PackageManager`
- `Uri.getScheme`
- `Uri.getHost`
- `takePersistableUriPermission`
- `getCanonicalPath`
- `startsWith`
- `baseDir`
- `getFilesDir`
- `FileProvider`
- `grantUriPermission`
- `FLAG_GRANT_READ_URI_PERMISSION`
- `FLAG_GRANT_WRITE_URI_PERMISSION`
- `MimeTypeMap`
- `allowlist`
- `validate`
- `UUID.randomUUID`
- `isFile`
- `isDirectory`

## 2.9 Android graph search recipes
Useful combinations:

```text
android:exported="true" + Uri.getPath + FileInputStream
ACTION_VIEW + getData + new File
ContentProvider.openFile + File
ContentProvider.delete + File.delete
Intent.EXTRA_STREAM + openInputStream + FileOutputStream
WebView.shouldInterceptRequest + FileInputStream
addJavascriptInterface + openFileInput
ZipInputStream + entry.getName + FileOutputStream
OpenableColumns.DISPLAY_NAME + File(parent, child)
getCanonicalPath without baseDir startsWith
```

---

# 3. Android Path Traversal Anti-Patterns

### A1. Exported deep link controls local file path
```java
Uri data = getIntent().getData();
File f = new File(getFilesDir(), data.getQueryParameter("file"));
return new FileInputStream(f);
```

Why risky:
An exported component or deep link may allow another app or browser to influence a local file path.

### A2. ContentProvider opens raw path from URI
```java
public ParcelFileDescriptor openFile(Uri uri, String mode) {
    return ParcelFileDescriptor.open(new File(baseDir, uri.getLastPathSegment()), MODE_READ_ONLY);
}
```

Why risky:
Provider URI path segments can escape the intended provider root if not canonicalized and contained.

### A3. WebView resource interception reads local file
```java
return new WebResourceResponse("text/plain", "utf-8", new FileInputStream(path));
```

Why risky:
URL-controlled resource paths can disclose local app files if URL-to-file mapping is weak.

### A4. Archive extraction trusts entry name
```java
File out = new File(destDir, entry.getName());
copy(zip, new FileOutputStream(out));
```

Why risky:
Archive entries can write outside the extraction directory.

---

# 4. Case Templates

## Case A-PATH-1: Exported component arbitrary read

### Vulnerable pattern
```java
File f = new File(getFilesDir(), getIntent().getStringExtra("name"));
return new FileInputStream(f);
```

### Audit focus
Verify exported reachability, caller controls, resolved base containment, and whether sensitive files can be read.

## Case A-PATH-2: ContentProvider arbitrary delete

### Vulnerable pattern
```java
new File(baseDir, uri.getLastPathSegment()).delete();
```

### Audit focus
Verify provider permissions, URI segment validation, canonical containment, and destructive effect.

## Case A-PATH-3: Unsafe archive extraction

### Vulnerable pattern
```java
File out = new File(destDir, zipEntry.getName());
```

### Audit focus
Verify entry path validation, overwrite behavior, and symlink handling.

---

# 5. Android-Specific Audit Heuristics

## 5.1 Exported reachability heuristics
Check whether the entry is reachable from another app, browser deep link, file share, push payload, or IPC caller.

## 5.2 Content URI heuristics
Pay attention to `ContentProvider.openFile`, `openAssetFile`, `query`, `delete`, and `FileProvider` mappings.

## 5.3 WebView heuristics
Treat URL-to-local-file mapping and JavaScript bridge file helpers as external input paths unless origin control is clearly proven.

## 5.4 Storage heuristics
Review app-private, external, cache, scoped-storage, and SAF paths separately because controls differ.
