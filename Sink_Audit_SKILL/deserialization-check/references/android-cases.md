# Android Unsafe Deserialization Cases

## Purpose

This file contains Android-specific unsafe deserialization patterns, candidate sink inventories, and audit cases.

Use it when the target application includes Android code, especially in:
- exported activities, services, broadcast receivers, or content providers
- deep links and app links
- WebView JavaScript bridges
- Binder, AIDL, Messenger, or Intent-based IPC
- push notification handlers
- background work, import, backup, restore, or file-processing flows
- Java/Kotlin serializers used inside Android apps

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify exported reachability, attacker-controlled input, object restoration behavior, dangerous trigger behavior, and missing platform or application controls.

---

# 1. Android Deserialization Control Points

## 1.1 Entry and IPC points
Look for:
- exported components
- intent filters
- deep links
- WebView bridges
- Binder or AIDL calls
- Parcelable or Serializable extras
- push notification payload handling
- file import, backup, restore, and share targets

## 1.2 Object restoration points
Look for:
- Intent and Bundle extra restoration
- Parcel and Parcelable restoration
- Serializable restoration
- Java native deserialization
- JSON/YAML/XML polymorphic mappers
- cache/session/token restore helpers
- SDK callback payload parsing

## 1.3 Object behavior controls
Look for:
- `Parcelable.Creator`
- `createFromParcel`
- `readFromParcel`
- `readObject`
- `readResolve`
- class loader usage
- post-restore trust in object state

## 1.4 Trust-boundary controls
Look for:
- `android:exported`
- component permissions
- signature permissions
- package/caller validation
- action/scheme/host validation
- typed extras and DTOs
- class allowlists
- integrity protection on stored blobs

---

# 2. High-Coverage Android Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker reachability and the actual restore behavior.

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
- `android:mimeType`
- `android:permission`
- `android:grantUriPermissions`
- `android:readPermission`
- `android:writePermission`
- `android:authorities`

## 2.2 Activity, service, receiver, provider, and deep-link entries
Search for:
- `onCreate`
- `onNewIntent`
- `onActivityResult`
- `registerForActivityResult`
- `getIntent`
- `getAction`
- `getData`
- `getExtras`
- `onStartCommand`
- `onBind`
- `onHandleIntent`
- `JobIntentService`
- `WorkManager`
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
- `call`
- `FirebaseMessagingService`
- `onMessageReceived`
- `ShareCompat`
- `ACTION_VIEW`
- `ACTION_SEND`
- `ACTION_SEND_MULTIPLE`

## 2.3 WebView, bridge, and IPC entries
Search for:
- `WebView`
- `addJavascriptInterface`
- `@JavascriptInterface`
- `shouldOverrideUrlLoading`
- `shouldInterceptRequest`
- `evaluateJavascript`
- `loadUrl`
- `postMessage`
- `WebMessagePort`
- `onTransact`
- `Binder`
- `IBinder`
- `IInterface`
- `AIDL`
- `Messenger`
- `Handler`
- `Message`
- `ResultReceiver`
- `PendingIntent`
- `Intent.parseUri`
- `sendBroadcast`
- `startActivity`
- `startService`
- `bindService`

## 2.4 Android object-restoration sink candidates
Search for:
- `getSerializableExtra`
- `getParcelableExtra`
- `getParcelableArrayExtra`
- `getParcelableArrayListExtra`
- `getBundleExtra`
- `getExtras`
- `Bundle.getSerializable`
- `Bundle.getParcelable`
- `Bundle.getParcelableArrayList`
- `Bundle.readFromParcel`
- `Parcel.readSerializable`
- `Parcel.readParcelable`
- `Parcel.readBundle`
- `Parcel.readArrayList`
- `Parcel.readMap`
- `Parcelable.Creator`
- `createFromParcel`
- `readFromParcel`
- `writeToParcel`
- `setClassLoader`
- `BadParcelableException`
- `ClassLoader`
- `Serializable`
- `ObjectInputStream`
- `readObject`
- `Base64.decode`
- `GZIPInputStream`

## 2.5 Java/Kotlin serializer sink candidates in Android
Search for:
- `ObjectInputStream.readObject`
- `SerializationUtils.deserialize`
- `ObjectMapper.readValue`
- `activateDefaultTyping`
- `enableDefaultTyping`
- `JsonTypeInfo`
- `LaissezFaireSubTypeValidator`
- `JSON.parseObject`
- `Feature.SupportAutoType`
- `ParserConfig.setAutoTypeSupport`
- `Gson.fromJson`
- `RuntimeTypeAdapterFactory`
- `XStream.fromXML`
- `Yaml.load`
- `Kryo.readClassAndObject`
- `HessianInput.readObject`
- `Moshi.adapter`
- `PolymorphicJsonAdapterFactory`
- `kotlinx.serialization`
- `Json.decodeFromString`
- `sealedSubclasses`
- `classDiscriminator`

## 2.6 Trigger and gadget behavior candidates
Search for:
- `Parcelable.Creator`
- `createFromParcel`
- `readFromParcel`
- `readObject`
- `readResolve`
- `readExternal`
- `onCreate`
- `onStartCommand`
- `onReceive`
- `ContentProvider.call`
- `Class.forName`
- `newInstance`
- `Method.invoke`
- `DexClassLoader`
- `PathClassLoader`
- `Runtime.getRuntime`
- `ProcessBuilder`
- `loadUrl`
- `evaluateJavascript`
- `startActivity`
- `sendBroadcast`

## 2.7 Required-control candidates
Search near sinks for:
- `android:exported="false"`
- `android:permission`
- `signature`
- `checkCallingPermission`
- `checkCallingOrSelfPermission`
- `getCallingPackage`
- `getCallingUid`
- `PackageManager`
- `resolveActivity`
- `setPackage`
- `setComponent`
- `setClassLoader`
- typed `getParcelableExtra`
- `requireNotNull`
- `allowed`
- `allowlist`
- `validate`
- `Uri.getScheme`
- `Uri.getHost`
- `Digital Asset Links`
- `HMAC`
- `Mac`
- `Signature`

## 2.8 Android graph search recipes
Useful combinations:

```text
android:exported="true" + getSerializableExtra
ACTION_VIEW + getParcelableExtra
onReceive + Bundle.getSerializable
onStartCommand + readParcelable
addJavascriptInterface + deserialize
onTransact + readSerializable
ContentProvider.call + Bundle.getParcelable
FirebaseMessagingService + JSON.parseObject
getExtras + setClassLoader missing
Parcel.readSerializable + exported component
```

---

# 3. Android Unsafe Deserialization Anti-Patterns

### A1. Exported component reads Serializable extra
```java
Serializable obj = getIntent().getSerializableExtra("payload");
```

Why risky:
An exported component may allow another app to influence restored object state.

### A2. Parcelable restored from untrusted IPC without type controls
```java
MyParcelable value = intent.getParcelableExtra("value");
```

Why risky:
Parcelable construction logic can be triggered from attacker-supplied parcels or extras when the component is reachable.

### A3. Missing class loader before Bundle unparceling
```java
Bundle b = intent.getExtras();
Object value = b.get("payload");
```

Why risky:
Class loader confusion or unexpected class restoration may occur when bundles cross process boundaries.

### A4. Deep link or WebView bridge reaches object mapper
```java
Object obj = mapper.readValue(uri.getQueryParameter("state"), Object.class);
```

Why risky:
Deep links and JavaScript bridges are attacker-influenced entry points when not tightly constrained.

---

# 4. Case Templates

## Case A-DESER-1: Exported Activity Serializable extra

### Vulnerable pattern
```java
Serializable payload = getIntent().getSerializableExtra("payload");
```

### Audit focus
Verify whether the activity is exported or deep-link reachable, whether the extra is attacker-controlled, and whether restored object state triggers sensitive behavior.

## Case A-DESER-2: Binder or ContentProvider Bundle restoration

### Vulnerable pattern
```java
Bundle b = extras.getBundle("data");
Object obj = b.getSerializable("object");
```

### Audit focus
Verify caller identity, permission checks, class loader handling, and type restrictions.

## Case A-DESER-3: WebView bridge to polymorphic mapper

### Vulnerable pattern
```java
@JavascriptInterface
public void restore(String payload) {
    mapper.readValue(payload, Object.class);
}
```

### Audit focus
Verify bridge exposure, loaded origins, mapper configuration, and post-restore effects.

---

# 5. Android-Specific Audit Heuristics

## 5.1 Exported reachability heuristics
Check whether the entry is reachable from another app, browser deep link, push payload, file share, or IPC caller.

## 5.2 Intent and Bundle heuristics
Pay attention to Serializable extras, Parcelable extras, missing `setClassLoader`, and object state trusted after restoration.

## 5.3 WebView bridge heuristics
Treat JavaScript bridge methods as external entry points unless origin control is clearly proven.

## 5.4 IPC consistency heuristics
Compare activity, service, receiver, provider, Binder, and background paths for inconsistent validation before restoration.
