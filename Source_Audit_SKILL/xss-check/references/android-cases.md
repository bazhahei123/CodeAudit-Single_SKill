# Android XSS Source Cases

## Purpose

This file contains Android-specific source point patterns and candidate search terms for XSS source discovery.

Use it when Android code accepts mobile-controlled or external content and then renders it through WebView, JavaScript bridges, HTML/rich-text widgets, markdown renderers, notifications, hybrid views, or backend-fed native screens, especially in:
- exported activities, services, receivers, and content providers
- deep links, app links, share targets, push callbacks, SDK callbacks, and IPC handlers
- WebView screens, URL interception, `loadData`, `loadUrl`, and JavaScript bridges
- `Html.fromHtml`, `TextView` rich text, markdown renderers, Compose wrappers, and notification previews

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify exported reachability, caller trust, source propagation, downstream rendering relevance, and later context-specific controls.

---

# 1. High-Coverage Android XSS Source Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; keep it only when code evidence shows that the value can influence WebView content, native HTML/rich-text rendering, JavaScript execution, markdown conversion, notification HTML, component state, sanitizer output, or backend-fed hybrid rendering.

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
- `android:mimeType`
- `android:authorities`
- `android:permission`
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
- `getCharSequenceExtra`
- `Intent.getAction`
- `Uri.getQueryParameter`
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
- `call`
- `FirebaseMessagingService`
- `onMessageReceived`
- `RemoteMessage.getData`
- `Worker.doWork`
- `Data.getString`
- SDK callback payloads
- notification payloads

## 1.3 WebView, bridge, and IPC source APIs

Search for:
- `WebView`
- `addJavascriptInterface`
- `@JavascriptInterface`
- `evaluateJavascript`
- `loadUrl`
- `loadData`
- `loadDataWithBaseURL`
- `shouldOverrideUrlLoading`
- `shouldInterceptRequest`
- `WebResourceRequest`
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
- `Bundle`
- `ResultReceiver`
- `ContentResolver.call`

## 1.4 Reflected and stored content source candidates

Search for Intent, Uri, Bundle, WebView bridge, provider, worker, or SDK fields named:
- `html`
- `rawHtml`
- `safeHtml`
- `trustedHtml`
- `bodyHtml`
- `contentHtml`
- `messageHtml`
- `markdown`
- `richText`
- `title`
- `message`
- `body`
- `content`
- `description`
- `label`
- `displayName`
- `username`
- `comment`
- `chat`
- `notification`
- `preview`
- `url`
- `href`
- `src`
- `script`
- `template`

## 1.5 Native HTML, markdown, and sanitizer source candidates

Search for source values near:
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
- `DOMPurify` in hybrid layer
- server-side sanitizer result
- allowed tags
- allowed attributes
- preview renderer
- final renderer
- notification builder
- Compose state
- `AndroidView`
- `ComposeView`
- `rememberWebViewState`

## 1.6 Downstream rendering relevance mapping candidates

After finding a source candidate, trace toward:
- `WebView.loadData`
- `WebView.loadDataWithBaseURL`
- `WebView.loadUrl`
- `evaluateJavascript`
- `javascript:`
- `addJavascriptInterface`
- `@JavascriptInterface`
- `Html.fromHtml`
- `HtmlCompat.fromHtml`
- `TextView.setText`
- `Markwon.setMarkdown`
- `LinkMovementMethod`
- notification text builders
- `setContent`
- `AndroidView`
- WebView wrapper components
- backend API calls that return HTML to WebView/native views

## 1.7 Stored, background, and second-order Android XSS sources

Search for:
- stored chat or comment records
- push payloads stored for later display
- SDK callback content cached locally
- WorkManager input rendered later
- local database records containing HTML, markdown, or rich text
- preference values used as render text
- downloaded config controlling HTML templates
- WebView bridge inputs persisted before rendering
- content provider writes later used by notifications or WebView

## 1.8 Android graph search recipes

Useful combinations:

```text
android:exported="true" + getStringExtra/getData + html/message/content
ACTION_VIEW/ACTION_SEND + Uri.getQueryParameter + WebView.loadData/loadDataWithBaseURL
@JavascriptInterface + callback/message/html + evaluateJavascript/loadUrl("javascript:")
RemoteMessage.getData + notification body + Html.fromHtml/TextView.setText
Worker.doWork + stored markdown/html + Markwon/WebView/TextView
ContentProvider.query/call + contentHtml/bodyHtml + native render path
Html.fromHtml/HtmlCompat.fromHtml + Intent/Bundle/stored content
WebView.setJavaScriptEnabled(true) + mobile-controlled HTML or URL
```

---

# 2. Android Source Patterns

## A-S1. Exported component supplies render content

Example idea:
- an exported activity, service, receiver, provider, or deep link reads `html`, `message`, `body`, `content`, `markdown`, or `url` and passes it toward WebView or native HTML rendering.

Audit relevance:
The source is mobile/client-controlled when another app, browser, deep link, share target, push payload, or IPC caller can supply it.

Follow-up:
- verify export status, caller checks, rendering context, JavaScript settings, and sanitization/escaping.

## A-S2. WebView bridge or JavaScript source

Example idea:
- JavaScript bridge, WebView message, intercepted URL, or `javascript:` call receives dynamic content that is rendered or executed in WebView.

Audit relevance:
WebView sources require origin checks, bridge exposure review, and final rendering context verification.

Follow-up:
- inspect WebView origin restrictions, bridge registration, JavaScript settings, and downstream render behavior.

## A-S3. Push, SDK, or worker content source

Example idea:
- push payload, SDK callback content, or worker input contains title/body/html/markdown displayed in notifications, views, or previews.

Audit relevance:
These are external-system-controlled or stored attacker-influenced unless origin and integrity are proven.

Follow-up:
- identify writer path, storage path, notification path, preview path, and final display controls.

## A-S4. Native rich text or markdown source

Example idea:
- content is converted through `Html.fromHtml`, Markwon, markdown renderers, or rich text widgets before display.

Audit relevance:
Native rich text can interpret markup, links, spans, and URL-like content differently from plain text.

Follow-up:
- verify sanitizer configuration, allowed tags/links, URL schemes, and preview/final consistency.

---

# 3. False-Positive Controls

Do not mark an Android source as high-priority if:
- the component is not externally reachable and caller trust is proven,
- values are fixed server-side or rendered only as plain text,
- the value affects only logs, analytics, or non-rendering metadata,
- the value never reaches WebView, JavaScript, native HTML widgets, markdown/rich-text renderers, notifications, or backend-fed hybrid rendering,
- stored content is trusted-only and cannot be attacker-influenced.

Use `Suspected source` or `Not enough evidence` if:
- export status, caller identity, or WebView origin control is unclear,
- sanitizer or render wrapper behavior is hidden,
- JavaScript settings or bridge exposure are not visible,
- stored writer paths are missing.

---

# 4. Quick Android Source Checklist

- Are exported components, deep links, share targets, providers, WebView bridges, SDK callbacks, push handlers, or IPC methods feeding render content?
- Do Intent, Uri, Bundle, message, worker, or WebView values influence HTML, markdown, rich text, script, URL, notification, or preview content?
- Are WebView bridge and URL interception sources origin-restricted?
- Are saved comments, notifications, markdown, or HTML values reused by background jobs or native views?
- Is the source only rendered as plain text, or can it become WebView/native HTML/rich text?
