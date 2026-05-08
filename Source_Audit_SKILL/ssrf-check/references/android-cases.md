# Android SSRF Source Cases

## Purpose

This file contains Android-specific source point patterns and candidate search terms for SSRF source discovery.

Use it when Android code accepts mobile-controlled request targets or request-target components and then makes app-side requests, drives WebView/network SDK behavior, or forwards targets to backend/network services, especially in:
- exported activities, services, receivers, and content providers
- deep links, app links, share targets, push callbacks, and SDK callbacks
- WebView JavaScript bridges and URL interception
- Binder, AIDL, Messenger, ResultReceiver, and Intent-based IPC
- mobile proxy, preview, import, webhook, diagnostics, storage SDK endpoint, and cloud SDK endpoint logic

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify exported reachability, caller trust, source propagation, downstream outbound-request relevance, and later target controls.

---

# 1. High-Coverage Android SSRF Source Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; keep it only when code evidence shows that the value can influence request-target construction, URL parsing, URL recomposition, host/scheme/port/path selection, redirect behavior, proxy/client options, SDK endpoint overrides, WebView navigation, or outbound request wrapper behavior.

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
- `<category`
- `<data`
- `android:scheme`
- `android:host`
- `android:path`
- `android:pathPrefix`
- `android:pathPattern`
- `android:authorities`
- `android:permission`
- `android:readPermission`
- `android:writePermission`
- `android:grantUriPermissions`
- `ACTION_VIEW`
- `ACTION_SEND`
- app links
- deep links
- share targets

## 1.2 Activity, service, receiver, provider, and callback source APIs

Search for:
- `onCreate`
- `onNewIntent`
- `getIntent`
- `getData`
- `getExtras`
- `getStringExtra`
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
- `loadUrl`
- `shouldOverrideUrlLoading`
- `shouldInterceptRequest`
- `WebResourceRequest`
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

## 1.4 Direct target and URL source candidates

Search for Intent, Uri, Bundle, WebView bridge, provider, message, worker, or SDK fields named:
- `url`
- `uri`
- `target`
- `targetUrl`
- `remoteUrl`
- `externalUrl`
- `callback`
- `callbackUrl`
- `webhook`
- `webhookUrl`
- `redirectUrl`
- `returnUrl`
- `previewUrl`
- `imageUrl`
- `avatarUrl`
- `fileUrl`
- `downloadUrl`
- `importUrl`
- `feedUrl`
- `sitemapUrl`
- `metadataUrl`
- `endpoint`
- `baseUrl`
- `serviceUrl`
- `providerUrl`
- `tenantUrl`

## 1.5 Partial destination, protocol, and client-option candidates

Search for:
- `host`
- `hostname`
- `domain`
- `ip`
- `address`
- `scheme`
- `protocol`
- `port`
- `path`
- `query`
- `resource`
- `endpointOverride`
- `proxyHost`
- `proxyUrl`
- `proxy`
- `redirect`
- `followRedirects`
- `followSslRedirects`
- `timeout`
- `ssl`
- `tls`
- `trustAllCerts`

## 1.6 URL construction, parser, and decoding candidates

Search for source values near:
- `Uri.parse`
- `Uri.Builder`
- `URL`
- `URI`
- `URLDecoder.decode`
- `Base64.decode`
- `toHttpUrl`
- `HttpUrl.parse`
- `HttpUrl.Builder`
- `Uri.getScheme`
- `Uri.getHost`
- `Uri.getPort`
- `Uri.getPath`
- string concatenation around `http://` or `https://`
- `InetAddress.getByName`
- `InetAddress.getAllByName`
- custom resolver helpers

## 1.7 Downstream SSRF relevance mapping candidates

After finding a source candidate, trace toward:
- `OkHttpClient`
- `Request.Builder.url`
- `Call.execute`
- `Call.enqueue`
- `Retrofit.Builder.baseUrl`
- `HttpURLConnection`
- `URL.openConnection`
- `URL.openStream`
- `Volley.newRequestQueue`
- `StringRequest`
- `JsonObjectRequest`
- `CronetEngine`
- `UrlRequest.Builder`
- Ktor `HttpClient`
- `Fuel`
- `Ion.with`
- `Glide.load`
- `Picasso.load`
- `Coil`
- `WebView.loadUrl`
- `shouldInterceptRequest`
- cloud SDK endpoint override builders
- backend execution or fetch APIs that receive the target

## 1.8 Stored, background, and second-order Android SSRF sources

Search for:
- stored webhook or callback targets
- saved preview targets
- synced remote endpoint settings
- push payload URLs stored for later fetch
- WorkManager input URLs
- local database records containing endpoints
- preference values used as base URLs or proxy settings
- downloaded config controlling endpoints
- WebView bridge inputs persisted before request use
- content provider writes later used by network jobs
- cloud storage endpoint overrides

## 1.9 Android graph search recipes

Useful combinations:

```text
android:exported="true" + getStringExtra/getData + url/host/endpoint
ACTION_VIEW + Uri.getQueryParameter + Request.Builder.url/Retrofit baseUrl
@JavascriptInterface + callbackUrl/webhookUrl/endpoint + OkHttp/WebView/backend API
shouldOverrideUrlLoading/shouldInterceptRequest + user URL + WebView/network client
onReceive/onStartCommand + RemoteMessage/Intent URL + HttpURLConnection
Worker.doWork + stored URL/callback + OkHttp/Retrofit
Glide.load/Picasso.load/Coil + Intent/Bundle imageUrl
endpointOverride/baseUrl + Intent/stored config + cloud SDK/client builder
```

---

# 2. Android Source Patterns

## A-S1. Exported component supplies request target

Example idea:
- an exported activity, service, receiver, provider, or deep link reads `url`, `host`, `callbackUrl`, `imageUrl`, or `endpoint` and passes it toward a network client or backend fetch API.

Audit relevance:
The source is mobile/client-controlled when another app, browser, deep link, push payload, or IPC caller can supply it.

Follow-up:
- verify export status, caller identity checks, target allowlists, final resolved-address checks, and redirect behavior.

## A-S2. WebView bridge or URL interception source

Example idea:
- JavaScript bridge, `loadUrl`, or `shouldInterceptRequest` receives a URL or host and drives app-side fetches, WebView navigation, or backend request helpers.

Audit relevance:
WebView sources require origin checks, bridge exposure review, and final target validation.

Follow-up:
- inspect WebView origin restrictions, bridge registration, and downstream network behavior.

## A-S3. SDK callback, push, or WorkManager target source

Example idea:
- SDK callback data, push payload data, or worker input contains callback, preview, import, or endpoint values later fetched by app code.

Audit relevance:
These are external-system-controlled or stored attacker-influenced unless origin and integrity are proven.

Follow-up:
- identify writer path, storage path, and revalidation before fetch.

## A-S4. Dynamic client option or endpoint override source

Example idea:
- mobile input, stored config, or tenant config controls `baseUrl`, endpoint override, proxy, redirect behavior, or scheme.

Audit relevance:
Client options can expand the reachable destination or route traffic through attacker-controlled infrastructure.

Follow-up:
- verify fixed endpoints, trusted config origins, and constrained proxy/protocol behavior.

---

# 3. False-Positive Controls

Do not mark an Android source as high-priority if:
- the component is not externally reachable and caller trust is proven,
- values are fixed server-side or mapped to strict endpoint allowlists,
- the value affects only UI display, analytics labels, logs, or non-network metadata,
- the value never reaches URL construction, WebView navigation, network clients, SDK endpoint builders, or backend fetch APIs,
- stored endpoint values are trusted-only and cannot be attacker-influenced.

Use `Suspected source` or `Not enough evidence` if:
- export status, caller identity, or WebView origin control is unclear,
- a network wrapper or backend API hides final request behavior,
- redirects, DNS checks, or endpoint override behavior are not visible,
- stored writer paths are missing.

---

# 4. Quick Android Source Checklist

- Are exported components, deep links, providers, WebView bridges, SDK callbacks, push handlers, or IPC methods feeding request targets?
- Do Intent, Uri, Bundle, message, worker, or WebView values influence URL, host, scheme, port, path, proxy, redirect, or endpoint override?
- Are WebView bridge and URL interception sources origin-restricted?
- Are saved callback, preview, import, or endpoint values reused by background jobs?
- Is the source only mapped to a fixed trusted endpoint, or can it alter the final request target?
