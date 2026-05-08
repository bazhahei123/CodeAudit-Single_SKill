# Android Access Control Cases

## Purpose

This file contains Android-specific access control sink candidates, anti-patterns, and audit cases.

Use it when the target application includes Android code, especially:
- exported activities, services, broadcast receivers, and content providers
- deep links and app links
- Binder, AIDL, Messenger, PendingIntent, and cross-app IPC
- WebView JavaScript bridges
- file sharing through `FileProvider`
- account, payment, admin, enterprise, or tenant-aware mobile workflows

This reference is guidance, not proof. Do not report a vulnerability only because a component resembles one of the candidates below. Always verify exported status, caller identity, permission checks, object ownership, and backend enforcement in the target code.

---

# 1. Android Authorization Control Points

## 1.1 Exported component sink candidates
Look for:
- `AndroidManifest.xml`
- `<activity>`
- `<service>`
- `<receiver>`
- `<provider>`
- `android:exported="true"`
- `android:exported` missing on components with `intent-filter`
- `<intent-filter>`
- custom actions
- deep links with `<data android:scheme=... android:host=...>`
- app links with `android:autoVerify="true"`
- `android:permission`
- `android:readPermission`
- `android:writePermission`
- `android:grantUriPermissions`
- `android:authorities`

Questions:
- Is this component reachable by another app?
- Does it expose sensitive data or actions?
- Is access restricted by manifest permission and runtime caller validation?

## 1.2 Activity and deep-link candidates
Look for:
- `Activity#onCreate`
- `Activity#onNewIntent`
- `getIntent()`
- `Intent.getData()`
- `getStringExtra`, `getParcelableExtra`, `getSerializableExtra`
- `NavDeepLinkBuilder`
- Jetpack Navigation deep links
- custom URL schemes
- auth reset, invite, payment, approval, export, or admin deep links

Questions:
- Can another app invoke the activity or deep link?
- Are sensitive actions executed from intent extras or URI parameters?
- Is the logged-in account, tenant, object owner, or workflow state checked before action execution?

## 1.3 Service, Binder, and AIDL candidates
Look for:
- `Service#onStartCommand`
- `IntentService`
- `JobIntentService`
- `bindService`
- `onBind`
- `Binder`
- `AIDL`
- `Messenger`
- `ResultReceiver`
- `PendingIntent`
- `startForegroundService`
- `JobService`
- `WorkManager` tasks triggered by external input

Questions:
- Can another app start or bind to the service?
- Does the service perform privileged local or backend actions?
- Does it check `Binder.getCallingUid()`, signature permission, package identity, or trusted caller state?

## 1.4 Broadcast receiver candidates
Look for:
- `BroadcastReceiver#onReceive`
- static receivers in manifest
- dynamic receivers through `registerReceiver`
- custom broadcast actions
- ordered broadcasts
- implicit broadcasts
- `LocalBroadcastManager` replacement patterns
- receiver-triggered delete, sync, export, reset, approve, or account actions

Questions:
- Can another app send the broadcast?
- Is the broadcast protected by sender/receiver permission?
- Does receiver logic trust extras without validating caller and current user state?

## 1.5 ContentProvider and file sharing candidates
Look for:
- `ContentProvider#query`
- `insert`, `update`, `delete`
- `openFile`, `openAssetFile`
- `call`
- `ContentResolver`
- `FileProvider`
- `DocumentsProvider`
- `android:grantUriPermissions`
- URI permissions
- path permissions
- provider authorities

Questions:
- Can another app read, write, update, delete, or open files?
- Are row/object IDs scoped to the current user/account/tenant?
- Are URI grants narrow and tied to intended recipients?

## 1.6 WebView and bridge candidates
Look for:
- `WebView`
- `addJavascriptInterface`
- `shouldOverrideUrlLoading`
- `shouldInterceptRequest`
- `setJavaScriptEnabled(true)`
- `loadUrl`
- `evaluateJavascript`
- custom scheme handlers
- bridge methods annotated with `@JavascriptInterface`

Questions:
- Can untrusted web content call privileged native methods?
- Are bridge actions bound to current authenticated user and backend authorization?
- Are URL loads and custom schemes restricted?

## 1.7 High-coverage Android sink candidate inventory

Use these search candidates to find Android access-control sinks quickly:
- `android:exported`
- `intent-filter`
- `android:permission`
- `android:readPermission`
- `android:writePermission`
- `getIntent()`
- `onNewIntent`
- `getData()`
- `getExtras()`
- `startActivity`
- `startService`
- `bindService`
- `sendBroadcast`
- `onReceive`
- `onStartCommand`
- `onBind`
- `Binder.getCallingUid`
- `checkCallingPermission`
- `checkCallingOrSelfPermission`
- `enforceCallingPermission`
- `PackageManager#getPackagesForUid`
- `ContentProvider#query`
- `ContentProvider#insert`
- `ContentProvider#update`
- `ContentProvider#delete`
- `openFile`
- `FileProvider`
- `addJavascriptInterface`
- `@JavascriptInterface`
- `PendingIntent`

---

# 2. Android Access Control Anti-Patterns

## A1. Exported component performs sensitive action without caller validation
Why risky:
Any other app may be able to trigger privileged local or backend behavior.

Verify:
- exported status,
- manifest permissions,
- runtime caller checks,
- backend authorization before final action.

## A2. Deep link trusts object ID or action from URI
Why risky:
Attackers may craft a link that opens another user's object or triggers a restricted workflow.

Verify:
- object ownership,
- tenant/account binding,
- workflow state,
- server-side authorization.

## A3. ContentProvider exposes unscoped rows or files
Why risky:
Other apps may read or modify data by changing URI path, row ID, or selection.

Verify:
- provider export status,
- URI permissions,
- row/object scoping,
- read/write permission enforcement.

## A4. Binder or AIDL service trusts caller input
Why risky:
Cross-app callers may invoke privileged service methods if caller identity is not checked.

Verify:
- `Binder.getCallingUid()` checks,
- signature-level permissions,
- package allowlists,
- per-method authorization.

## A5. WebView bridge exposes privileged native method
Why risky:
Untrusted content may call native methods that access account data or backend APIs.

Verify:
- trusted origin,
- bridge method exposure,
- user/account binding,
- backend authorization.

---

# 3. Quick Android Audit Checklist

- Are exported components intentional and protected?
- Are deep links authorization-checked after parsing?
- Are content providers scoped by caller and object owner?
- Are Binder/AIDL methods checking calling UID and permission?
- Are broadcast receivers protected against external spoofing?
- Are PendingIntents immutable and scoped?
- Are WebView bridges exposed only to trusted content?
- Are backend calls still enforcing authorization even if mobile UI hides actions?
