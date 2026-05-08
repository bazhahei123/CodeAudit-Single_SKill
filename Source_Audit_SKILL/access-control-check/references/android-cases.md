# Android Access Control Source Cases

## Purpose

This file contains Android-specific source point patterns and audit cases for access-control source discovery.

Use it when the target application includes Android code, especially:
- exported activities, services, broadcast receivers, and content providers
- deep links and app links
- WebView JavaScript bridges and hybrid app entry points
- Binder, AIDL, Messenger, PendingIntent, and IPC handlers
- mobile API clients that pass identity, tenant, role, object, or workflow values into backend calls

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real source origin and downstream access-control relevance.

---

# 1. High-Coverage Android Source Candidate Inventory

## 1.1 Manifest and component entry sources

Look for:
- `android:exported="true"`
- `<activity>`
- `<service>`
- `<receiver>`
- `<provider>`
- `<intent-filter>`
- `<data android:scheme=...>`
- `<data android:host=...>`
- `android:permission`
- `android:readPermission`
- `android:writePermission`
- `android:grantUriPermissions`
- `FileProvider`
- dynamic receiver registration

Source questions:
- Can another app invoke the component or send data into it?
- Does the component accept user, tenant, role, object, action, or state values?
- Is caller identity verified before sensitive use?
- Does a mobile entry invoke a privileged backend API or local protected operation?

## 1.2 Intent, deep-link, and bundle source candidates

Look for:
- `getIntent()`
- `Intent.getData()`
- `Uri.getQueryParameter(...)`
- `getStringExtra(...)`
- `getIntExtra(...)`
- `getLongExtra(...)`
- `getParcelableExtra(...)`
- `getSerializableExtra(...)`
- `getExtras()`
- `Bundle.getString(...)`
- `Bundle.getLong(...)`
- `Bundle.getParcelable(...)`
- `onNewIntent(...)`
- `PendingIntent`
- `ActivityResultLauncher`
- `onActivityResult(...)`

High-priority fields:
- `user_id`, `userId`, `uid`, `account_id`, `member_id`
- `tenant_id`, `tenantId`, `org_id`, `workspace_id`, `company_id`
- `role`, `permission`, `scope`, `is_admin`, `access_level`
- `id`, `object_id`, `order_id`, `invoice_id`, `file_id`, `project_id`, `document_id`
- `action`, `operation`, `status`, `state`, `target_state`

## 1.3 Android identity and caller source candidates

Look for:
- `FirebaseAuth.getInstance().getCurrentUser()`
- `GoogleSignIn.getLastSignedInAccount(...)`
- `AccountManager`
- `BiometricPrompt` result handling
- app session/token stores
- `SharedPreferences` identity fields
- encrypted preference/session stores
- `getCallingUid()`
- `getCallingPackage()`
- `Binder.getCallingUid()`
- `PackageManager#getPackagesForUid`
- custom `UserSession`, `AuthManager`, `SessionManager`, `UserContext`, `TenantContext`

Source questions:
- Is identity from a verified session/token or an external caller?
- Are request parameters allowed to override stored identity?
- Are caller UID/package checks present for exported services/providers?

## 1.4 Authorization, permission, and component guard candidates

Look for:
- manifest `android:permission`, `readPermission`, `writePermission`
- `checkCallingPermission(...)`
- `checkCallingOrSelfPermission(...)`
- `enforceCallingPermission(...)`
- `ContextCompat.checkSelfPermission(...)`
- `AppOpsManager`
- `PackageManager.checkSignatures(...)`
- custom calls such as `canAccess`, `hasRole`, `hasPermission`, `isAdmin`, `isOwner`, `isTenantMember`
- backend request headers or JSON fields carrying `role`, `scope`, `tenant_id`, or `user_id`
- feature flags or entitlement checks used before privileged mobile actions

## 1.5 Object, tenant, and backend request source candidates

Look for:
- Retrofit annotations: `@GET`, `@POST`, `@PUT`, `@PATCH`, `@DELETE`, `@Path`, `@Query`, `@QueryMap`, `@Body`, `@Header`, `@Field`, `@Part`
- OkHttp request construction: `Request.Builder`, `url(...)`, `addHeader(...)`, JSON body builders
- Apollo/GraphQL variables and mutation input objects
- local database selectors: Room DAO `@Query`, `@Insert`, `@Update`, `@Delete`
- content provider selectors: `query`, `insert`, `update`, `delete`, `selection`, `selectionArgs`
- object IDs in extras, deep links, QR codes, NFC tags, push notifications, app widgets, shortcuts, and share targets
- tenant or organization values injected into backend calls

## 1.6 WebView and hybrid bridge source candidates

Look for:
- `addJavascriptInterface(...)`
- `@JavascriptInterface`
- `shouldOverrideUrlLoading(...)`
- `shouldInterceptRequest(...)`
- `onPageStarted(...)`
- `onPageFinished(...)`
- `evaluateJavascript(...)`
- `postMessage`
- deep link handlers invoked from WebView pages
- bridge methods that receive `userId`, `tenantId`, `role`, `token`, `objectId`, `action`, or `status`

Source questions:
- Can web content or another app control bridge parameters?
- Does the bridge trigger local privileged actions or authenticated backend APIs?
- Are origin, caller, and session checks visible?

## 1.7 Business action and workflow candidates

Look for method, route, or payload names containing:
- `approve`
- `reject`
- `publish`
- `archive`
- `delete`
- `disable`
- `enable`
- `lock`
- `unlock`
- `reset`
- `refund`
- `void`
- `cancel`
- `transfer`
- `assign`
- `share`
- `export`
- `download`
- `invite`
- `grant`
- `revoke`

Also inspect request fields:
- `action`
- `operation`
- `status`
- `state`
- `stage`
- `transition`
- `targetState`
- `targetStatus`

---

# 2. Android Source Patterns and Blind Spots

## 2.1 Exported component source

```java
public void onCreate(Bundle savedInstanceState) {
    String orderId = getIntent().getStringExtra("order_id");
    orderClient.cancel(orderId);
}
```

Why relevant:
`order_id` may be supplied by another app if the activity is exported or reachable through a deep link.

What to verify next:
- whether the component is exported or protected by permission
- whether caller identity or deep-link origin is verified
- whether backend authorization ignores client-controlled user/tenant values

## 2.2 Deep-link tenant source

```kotlin
val tenantId = intent.data?.getQueryParameter("tenant_id")
api.exportReport(tenantId, reportId)
```

Why relevant:
Deep-link parameters are client-controlled and may influence tenant-scoped backend operations.

What to verify next:
- whether tenant is derived from authenticated membership instead
- whether the backend rechecks tenant membership
- whether this path bypasses normal UI gating

## 2.3 JavaScript bridge action source

```java
@JavascriptInterface
public void runAction(String projectId, String action) {
    projectApi.runAction(projectId, action);
}
```

Why relevant:
Bridge arguments can become source points for object and business-action authorization review.

What to verify next:
- whether only trusted origins can call the bridge
- whether backend authorization checks object, action, and tenant
- whether the bridge is enabled in untrusted WebView content

---

# 3. Android Graph Search Recipes

```text
android:exported/intention-filter + getIntent/getExtra/getData + userId/tenantId/objectId/action
@JavascriptInterface/addJavascriptInterface + bridge argument + backend/local privileged action
ContentProvider query/update/delete + selection/Uri path segment + caller UID/package check
Binder/AIDL method + caller supplied id/tenant/action + enforceCallingPermission/checkCallingUid
Retrofit @Path/@Query/@Body + user_id/tenant_id/role/status + protected backend endpoint
```

---

# 4. False-Positive Controls

Do not mark an Android source as high-priority if:
- the component is not exported and no external or indirect trigger is visible,
- the value is overwritten by verified session or membership context before sensitive use,
- the value only affects display, sorting, pagination, or local non-sensitive UI state,
- the backend clearly treats mobile values as selectors and performs its own authorization.

Use `Suspected source` or `Not enough evidence` if:
- manifest reachability is unclear,
- bridge origin restrictions are hidden,
- caller verification happens in a helper that has not been inspected,
- backend enforcement is outside the visible code path.

---

# 5. Quick Android Source Checklist

- Are exported components receiving Intent extras, deep-link parameters, or URI path segments?
- Are WebView bridge methods receiving object, tenant, role, user, action, or state values?
- Are content provider queries or updates scoped by caller-controlled URI or selection values?
- Are Binder/AIDL methods checking caller UID/package or permissions before sensitive use?
- Are mobile API calls sending user, tenant, role, permission, or action values that should be server-derived?
- Are push notification, QR/NFC, app widget, shortcut, or share-target payloads driving protected actions?
