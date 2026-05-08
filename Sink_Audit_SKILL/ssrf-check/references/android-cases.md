# Android SSRF Cases

## Purpose

This file contains Android-specific SSRF and unsafe outbound request patterns, candidate sink inventories, and audit cases.

Use it when Android app code performs network requests from attacker-influenced mobile entry points, especially in:
- exported activities, services, receivers, and content providers
- deep links, app links, share targets, push callbacks, and SDK callbacks
- WebView JavaScript bridges and URL interception
- Binder, AIDL, Messenger, and Intent-based IPC
- mobile proxy, preview, import, webhook, diagnostics, or cloud-SDK endpoint logic

This reference is guidance, not proof. Confirm exported reachability, attacker-controlled target input, final request target, sink behavior, and missing controls.

---

# 1. Android SSRF Control Points

## 1.1 Entry and IPC points
Look for:
- exported components
- intent filters and deep links
- WebView bridge methods
- Binder/AIDL/Messenger calls
- ContentProvider methods
- push notification callbacks
- background work and sync jobs

## 1.2 Request sink points
Look for:
- OkHttp, HttpURLConnection, Retrofit, Volley, Cronet, Ktor, and WebView fetch logic
- image loaders and media fetchers
- cloud SDK clients with endpoint overrides
- WebView URL navigation or interception

---

# 2. High-Coverage Android SSRF Candidate Inventory

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
- `android:authorities`

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
- `call`
- `FirebaseMessagingService`
- `onMessageReceived`
- `ACTION_VIEW`
- `ACTION_SEND`

## 2.3 WebView, bridge, and IPC entries
Search for:
- `WebView`
- `addJavascriptInterface`
- `@JavascriptInterface`
- `loadUrl`
- `evaluateJavascript`
- `shouldOverrideUrlLoading`
- `shouldInterceptRequest`
- `WebResourceRequest`
- `onTransact`
- `Binder`
- `IBinder`
- `AIDL`
- `Messenger`
- `Handler`

## 2.4 Android outbound request sink candidates
Search for:
- `OkHttpClient`
- `Request.Builder.url`
- `Call.execute`
- `enqueue`
- `Retrofit.Builder.baseUrl`
- `HttpURLConnection`
- `URL.openConnection`
- `URL.openStream`
- `Volley.newRequestQueue`
- `StringRequest`
- `JsonObjectRequest`
- `CronetEngine`
- `UrlRequest.Builder`
- `Ktor HttpClient`
- `Fuel`
- `Ion.with`
- `Glide.load`
- `Picasso.load`
- `Coil`
- `WebView.loadUrl`

## 2.5 Request target construction candidates
Search for:
- `getQueryParameter`
- `getStringExtra`
- `Uri.parse`
- `Uri.Builder`
- `URLDecoder.decode`
- `Base64.decode`
- `toHttpUrl`
- `HttpUrl.parse`
- `HttpUrl.Builder`
- `baseUrl`
- `endpoint`
- `callbackUrl`
- `webhookUrl`
- `imageUrl`
- `host`
- `scheme`
- `port`

## 2.6 Required-control candidates
Search near sinks for:
- `android:exported="false"`
- `android:permission`
- `checkCallingPermission`
- `getCallingUid`
- `Uri.getScheme`
- `Uri.getHost`
- `InetAddress`
- `isLoopbackAddress`
- `isLinkLocalAddress`
- `isSiteLocalAddress`
- `169.254.169.254`
- `localhost`
- `127.0.0.1`
- `followRedirects(false)`
- `followSslRedirects(false)`
- host allowlist
- scheme allowlist

## 2.7 Android graph search recipes
Useful combinations:

```text
android:exported="true" + OkHttpClient
ACTION_VIEW + getQueryParameter + Request.Builder.url
@JavascriptInterface + Retrofit baseUrl
WebView.loadUrl + deep-link value
shouldInterceptRequest + user URL
onReceive + HttpURLConnection
Worker.doWork + stored URL
Glide.load + Intent extra
S3 endpointOverride + user/stored value
```

---

# 3. Android SSRF Anti-Patterns

### A1. Exported deep link controls OkHttp URL
```java
String url = getIntent().getData().getQueryParameter("url");
client.newCall(new Request.Builder().url(url).build()).execute();
```

Why risky:
Another app or browser can influence the server-side or app-side network target if the component is exported.

### A2. JavaScript bridge fetches arbitrary URL
```java
@JavascriptInterface
public void fetch(String url) {
    new OkHttpClient().newCall(new Request.Builder().url(url).build()).execute();
}
```

Why risky:
Untrusted web content may drive network fetches through the bridge.

### A3. Dynamic SDK endpoint
```java
S3Client.builder().endpointOverride(URI.create(endpoint)).build();
```

Why risky:
Attacker-controlled endpoint overrides can redirect SDK calls to unintended hosts.

---

# 4. Android-Specific Audit Heuristics

Review exported reachability, WebView origin controls, final host/IP restrictions, redirect handling, and SDK endpoint overrides.
