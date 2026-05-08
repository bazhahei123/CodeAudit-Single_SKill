# Android Unsafe Deserialization Source Cases

## Purpose

This file contains Android-specific source point patterns and candidate search terms for unsafe deserialization source discovery.

Use it when the target application includes Android code, especially:
- exported activities, services, broadcast receivers, and content providers
- deep links and app links
- WebView JavaScript bridges
- Binder, AIDL, Messenger, PendingIntent, and IPC handlers
- WorkManager jobs and background workers
- SDK callbacks, push notification handlers, and mobile API clients
- Android code that receives bundles, parcels, serialized extras, JSON/YAML/XML blobs, protobufs, or backend payloads before object restoration

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

## 1.2 Intent, Bundle, Parcel, and IPC source candidates

- `getIntent()`
- `Intent.getData()`
- `Uri.getQueryParameter(...)`
- `getStringExtra(...)`
- `getByteArrayExtra(...)`
- `getBundleExtra(...)`
- `getParcelableExtra(...)`
- `getSerializableExtra(...)`
- `getExtras()`
- `Bundle.getString(...)`
- `Bundle.getByteArray(...)`
- `Bundle.getParcelable(...)`
- `Bundle.getSerializable(...)`
- `Parcel`
- `readSerializable`
- `readParcelable`
- `readBundle`
- `readValue`
- `BaseBundle`
- `AIDL`
- `Messenger`
- `Binder.getCallingUid`
- `ContentValues`
- `Cursor`

## 1.3 Serialized payload and wrapper candidates

- `payload`
- `data`
- `body`
- `message`
- `blob`
- `bytes`
- `raw`
- `content`
- `serialized`
- `objectData`
- `parcel`
- `bundle`
- `proto`
- `protobuf`
- `base64`
- `gzip`
- `zip`
- `compressed`
- `encoded`
- `encrypted`
- `signed`
- `metadata`
- `config`
- `backup`
- `snapshot`
- push notification data
- QR/NFC/share payloads

## 1.4 Type and object restoration candidates

- `type`
- `class`
- `className`
- `objectType`
- `targetType`
- `kind`
- `discriminator`
- `Parcelable`
- `Serializable`
- `Externalizable`
- `CREATOR`
- `ClassLoader`
- `setClassLoader`
- `readSerializable`
- `readParcelable`
- `readBundle`
- `ObjectInputStream`
- `readObject`
- `Gson.fromJson`
- `Moshi.adapter`
- `Jackson ObjectMapper`
- `Yaml.load`
- protobuf parsers

## 1.5 Stored and second-order candidates

- `SharedPreferences`
- encrypted preferences
- Room database blobs
- SQLite blobs
- local files
- cache files
- WebView local storage
- downloaded config
- offline queue
- pending actions
- WorkManager input data
- push payload storage
- content-provider blobs
- object storage downloads
- SDK cached payloads

## 1.6 Android graph search recipes

```text
android:exported/deep link + getStringExtra/getByteArrayExtra + base64/deserialize
@JavascriptInterface + payload/data/class/type + Gson/Jackson/ObjectInputStream
Bundle/Parcel + getSerializable/readSerializable + ObjectInputStream/readObject
ContentProvider + blob/metadata + deserialize/parse/load
WorkManager/FirebaseMessagingService + payload/data + decode/restore
Retrofit/OkHttp response body + type/class metadata + object mapper
```

---

# 2. Android Source Patterns and Blind Spots

## A-S1. Exported component extras

Intent extras, deep-link parameters, or notification payload fields can become source points when they carry serialized blobs, type metadata, or object state.

Follow-up:
- verify component export status and caller controls,
- trace extras into parsers, object mappers, or backend payload construction.

## A-S2. Bundle, Parcelable, and Serializable sources

`Bundle`, `Parcel`, `Parcelable`, and `Serializable` data are source points when another app, WebView bridge, SDK, or IPC caller can influence them.

Follow-up:
- verify class loader constraints,
- identify custom `readObject`, `readParcelable`, or `readSerializable` behavior.

## A-S3. WebView bridge payloads

Bridge arguments can carry JSON, base64 blobs, class/type fields, or command-like object state.

Follow-up:
- verify WebView origin trust,
- trace bridge arguments into object mappers, local storage, or backend calls.

## A-S4. Stored mobile payloads

Local caches, offline queues, Room/SQLite blobs, and WorkManager input data can be second-order sources.

Follow-up:
- identify who can write the stored value,
- verify integrity and safe parser/mapper behavior on reload.

---

# 3. False-Positive Controls

Do not mark an Android source as high-priority if:
- the component is not externally reachable and no indirect trigger is visible,
- values are parsed only into strict primitives or DTOs with no object/type restoration relevance,
- parcelable/serializable data is constructed only by trusted app code,
- the downstream parser is visible and safe with strict schema/type controls.

Use `Suspected source` or `Not enough evidence` if:
- component reachability is unclear,
- class loader or parser configuration is hidden,
- WebView bridge origin restrictions are hidden,
- stored payload writer paths are missing.

---

# 4. Quick Android Source Checklist

- Are exported components receiving serialized extras, bundles, byte arrays, or deep-link blobs?
- Are WebView bridge methods receiving payload, class, type, or object state fields?
- Are `Parcelable` / `Serializable` / `Parcel` values influenced by IPC or external apps?
- Are local caches, Room/SQLite blobs, or WorkManager inputs later restored?
- Are mobile API responses or SDK callbacks mapped polymorphically?
- Are trigger-relevant fields such as path, URL, command, method, callback, or template stored in restored objects?
