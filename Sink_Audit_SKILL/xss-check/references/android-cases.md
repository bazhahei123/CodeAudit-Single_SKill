# Android XSS Cases

## Purpose

This file contains Android-specific XSS and browser-interpreted rendering patterns, candidate sink inventories, and audit cases.

Use it when Android app code displays attacker-influenced HTML, markdown, rich text, JavaScript, or web content, especially in:
- exported activities, services, receivers, providers, deep links, app links, push callbacks, and SDK callbacks
- WebView screens, WebView bridges, URL interception, and hybrid app views
- `Html.fromHtml`, rich text previews, markdown renderers, and native HTML widgets
- Compose or native UI wrappers around WebView or HTML-rendering components

This reference is guidance, not proof. Confirm exported reachability, attacker-controlled content, rendering context, sink behavior, and missing controls.

---

# 1. Android XSS Control Points

## 1.1 Entry and IPC points
Look for:
- exported components
- deep links and app links
- WebView bridge methods
- Binder/AIDL/Messenger handlers
- ContentProvider methods
- push notification callbacks
- SDK callback payloads
- background jobs that render stored content

## 1.2 Rendering points
Look for:
- WebView HTML/data loading
- JavaScript evaluation
- `Html.fromHtml`
- markdown/rich-text rendering
- notification, preview, chat, CMS, support, or admin views
- native UI wrappers that display HTML-like content

---

# 2. High-Coverage Android XSS Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows.

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
- `android:mimeType`
- `android:permission`

## 2.2 Activity, service, receiver, provider, and callback entries
Search for:
- `onCreate`
- `onNewIntent`
- `getIntent`
- `getData`
- `getExtras`
- `getStringExtra`
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
- `RemoteMessage`
- `notification`
- `preview`
- `chat`
- `message`

## 2.3 WebView and JavaScript sink candidates
Search for:
- `WebView`
- `loadData`
- `loadDataWithBaseURL`
- `loadUrl`
- `evaluateJavascript`
- `javascript:`
- `addJavascriptInterface`
- `@JavascriptInterface`
- `WebSettings.setJavaScriptEnabled(true)`
- `setAllowFileAccess`
- `setAllowContentAccess`
- `setAllowFileAccessFromFileURLs`
- `setAllowUniversalAccessFromFileURLs`
- `shouldOverrideUrlLoading`
- `shouldInterceptRequest`
- `WebViewClient`
- `WebChromeClient`
- `WebResourceResponse`

## 2.4 Native HTML, markdown, and rich-text sink candidates
Search for:
- `Html.fromHtml`
- `HtmlCompat.fromHtml`
- `TextView.setText`
- `Spanned`
- `SpannableString`
- `LinkMovementMethod`
- `Markdown`
- `Markwon`
- `commonmark`
- `flexmark`
- `RichEditor`
- `WYSIWYG`
- `bodyHtml`
- `contentHtml`
- `AndroidView`
- `ComposeView`
- `rememberWebViewState`
- `setContent`

## 2.5 Required-control candidates
Search near sinks for:
- `setJavaScriptEnabled(false)`
- no `addJavascriptInterface`
- origin allowlist
- `Uri.getScheme`
- `Uri.getHost`
- `WebViewAssetLoader`
- `shouldOverrideUrlLoading` host check
- `DOMPurify` in hybrid layer
- server-side sanitizer
- allowed tags
- allowed attributes
- URL scheme allowlist
- safe plain text rendering
- `TextView.setText` without `Html.fromHtml`
- preview/final render consistency

## 2.6 Android graph search recipes
Useful combinations:

```text
android:exported="true" + WebView.loadData
ACTION_VIEW + getQueryParameter + loadDataWithBaseURL
@JavascriptInterface + loadUrl("javascript:")
push notification body + Html.fromHtml
SDK callback + TextView.setText(Html.fromHtml)
Markwon + user markdown + WebView/TextView
WebView.setJavaScriptEnabled(true) + user HTML
addJavascriptInterface + untrusted WebView content
```

---

# 3. Android XSS Anti-Patterns

### A1. Deep link HTML loaded into WebView
```java
String html = getIntent().getData().getQueryParameter("html");
webView.loadDataWithBaseURL(null, html, "text/html", "UTF-8", null);
```

Why risky:
Attacker-controlled deep-link content is rendered as browser-interpreted HTML.

### A2. JavaScript bridge with untrusted WebView content
```java
webView.getSettings().setJavaScriptEnabled(true);
webView.addJavascriptInterface(this, "bridge");
webView.loadUrl(userUrl);
```

Why risky:
Untrusted web content can interact with native bridge methods.

### A3. Push or SDK payload rendered as HTML
```java
textView.setText(Html.fromHtml(remoteMessage.getData().get("body")));
```

Why risky:
Remote payloads may be interpreted as active links or unsafe markup depending on rendering context.

---

# 4. Android-Specific Audit Heuristics

Review exported reachability, WebView origin controls, JavaScript settings, JavaScript bridge exposure, HTML rendering widgets, markdown/rich-text renderers, and preview/final render consistency.
