# C++ XSS Source Cases

## Purpose

This file contains C++-specific source point patterns and candidate search terms for XSS source discovery.

Use it when the target application includes C++ or native service code that emits HTML, renders web UI, hosts embedded browsers, or generates previews, especially in:
- HTTP servers, CGI/FastCGI handlers, RPC/IPC handlers, WebSocket handlers, and native admin dashboards
- Crow, Drogon, oatpp, cpp-httplib, Pistache, Wt, CppCMS, Qt, CEF, WebView2, and template engines
- server-side HTML generation, native WebView rendering, markdown/rich-text preview, reports, and email HTML generation

This reference is guidance, not proof. C++ rendering code often hides source propagation behind string builders and wrappers. Always verify source origin, propagation, trust boundary, downstream rendering relevance, and later context-specific controls.

---

# 1. High-Coverage C++ XSS Source Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; keep it only when code evidence shows that the value can influence HTML responses, template data, native HTML widgets, embedded browser content, script data, attribute values, URL values, markdown/rich-text renderers, sanitizer output, or alternate render paths.

## 1.1 HTTP, RPC, WebSocket, CGI, and IPC entry candidates

Search for:
- `CROW_ROUTE`
- `crow::SimpleApp`
- `app.route_dynamic`
- `DROGON_BEGIN_NAMESPACE`
- `METHOD_LIST_BEGIN`
- `ADD_METHOD_TO`
- `HttpController`
- `HttpSimpleController`
- `HttpRequestPtr`
- `HttpResponsePtr`
- `oatpp::web::server::api::Controller`
- `ENDPOINT`
- `Pistache::Rest::Routes::Get`
- `Pistache::Rest::Routes::Post`
- `httplib::Server`
- `svr.Get`
- `svr.Post`
- `FastCGI`
- `FCGX`
- `CGI`
- `grpc::Service`
- `ServerContext`
- `onMessage`
- `WebSocket`
- `DBus`
- `Binder`
- `onTransact`
- custom IPC handlers

## 1.2 Native render, preview, admin, and message entries

Search for:
- native UI event handlers
- Qt slots
- CEF handlers
- WebView2 handlers
- admin dashboard
- report renderer
- email renderer
- preview renderer
- markdown preview
- CMS render
- support view
- chat view
- notification view
- queue consumer
- import/export renderer
- CLI/admin render tools

## 1.3 Reflected and stored content source candidates

Search for request, message, protobuf, JSON, CLI, config, or DTO fields named:
- `q`
- `query`
- `search`
- `keyword`
- `term`
- `message`
- `error`
- `reason`
- `title`
- `name`
- `displayName`
- `nickname`
- `username`
- `label`
- `description`
- `summary`
- `content`
- `body`
- `text`
- `comment`
- `reply`
- `review`
- `profile`
- `bio`
- `signature`
- `ticket`
- `notification`
- `announcement`
- `article`
- `post`
- `cms`

## 1.4 HTML, template, native UI, and API propagation candidates

Search for source values passed into:
- `text/html`
- `setContentType`
- `Content-Type: text/html`
- response body fields
- `res.write`
- `res.body`
- `setBody`
- `HttpResponse::newHttpResponse`
- `std::string html`
- `std::stringstream html`
- `std::ostringstream html`
- `fmt::format`
- `absl::StrFormat`
- `boost::format`
- `QString::arg`
- inja templates
- mustache templates
- Wt templates
- CppCMS templates
- API response JSON fields
- report/email template contexts

## 1.5 Raw HTML, native rich text, and embedded browser source candidates

Search for:
- `html`
- `rawHtml`
- `safeHtml`
- `trustedHtml`
- `bodyHtml`
- `contentHtml`
- `messageHtml`
- `rendered`
- `markdown`
- `richText`
- `wysiwyg`
- `template`
- `script`
- `href`
- `src`
- `style`
- `onclick`
- `Wt::WText`
- `Wt::TextFormat::XHTML`
- `Wt::TextFormat::XHTMLUnsafeText`
- `QTextDocument::setHtml`
- `QTextBrowser::setHtml`
- `QLabel::setText`
- `Qt::RichText`
- `QWebEngineView::setHtml`
- `QWebView::setHtml`
- `CefFrame::LoadString`
- `NavigateToString`

## 1.6 Markdown, sanitizer, and control pipeline candidates

Search for source values near:
- `markdown`
- `cmark`
- `md4c`
- `Discount`
- `Hoedown`
- `CommonMark`
- `sanitize`
- `escapeHtml`
- `htmlEscape`
- `QString::toHtmlEscaped`
- `Wt::Utils::htmlEncode`
- `Wt::TextFormat::Plain`
- allowed tags
- allowed attributes
- URL scheme allowlist
- preview renderer
- final renderer
- admin renderer
- email/web preview renderer

## 1.7 Downstream rendering relevance mapping candidates

After finding a source candidate, trace toward:
- response body writers
- `text/html` responses
- template renderers
- raw template variables
- manual HTML string builders
- Wt text widgets
- Qt rich text widgets
- Qt WebEngine
- CEF/WebView
- WebView2
- markdown-to-HTML renderers
- sanitizer wrappers
- JSON API serializers
- frontend consumers
- admin/report/email renderers

## 1.8 C++ graph search recipes

Useful combinations:

```text
CROW_ROUTE/svr.Get/ENDPOINT + query/message/content + text/html/setBody
CGI/FastCGI + request parameter + HTML response writer
Drogon/oatpp controller + stored comment/body + template render
std::stringstream/fmt::format/QString::arg + request/stored field + HTML output
inja/mustache/Wt template + rawHtml/contentHtml/safeHtml source
QTextDocument/QTextBrowser/QLabel + external content + RichText/setHtml
QWebEngineView/CEF/WebView2 + user/stored HTML + embedded browser render
markdown/cmark/md4c + stored body + HTML response/native widget
```

---

# 2. C++ Source Patterns

## CPP-S1. HTTP/RPC/IPC value becomes HTML response data

Example idea:
- HTTP query parameter, protobuf field, WebSocket message, IPC payload, or CLI argument becomes template data, HTML response text, report content, or native UI text.

Audit relevance:
The source may render safely or unsafely depending on template escaping, response context, and native widget interpretation.

Follow-up:
- trace into response writers, template engines, Wt/Qt/CEF/WebView wrappers, and frontend consumers.

## CPP-S2. Stored content display source

Example idea:
- comment, CMS body, profile field, ticket text, notification, report label, or imported content is rendered later in native or web UI.

Audit relevance:
Stored content can affect other users, admins, reports, previews, and embedded browser surfaces.

Follow-up:
- identify writer path, user/admin render paths, preview/final consistency, and sanitizer placement.

## CPP-S3. Raw HTML, rich text, or embedded browser source

Example idea:
- `rawHtml`, `safeHtml`, markdown output, `QString`, or `std::string html` reaches WebView, CEF, Qt rich text, Wt XHTML, or raw response output.

Audit relevance:
Native UI and embedded browser APIs may interpret markup instead of text.

Follow-up:
- verify escaping, sanitizer configuration, text-vs-rich format, URL scheme controls, and trusted wrapper origin.

## CPP-S4. Script, URL, attribute, or API-fed frontend source

Example idea:
- response data becomes inline script data, attributes, `href`, `src`, JSON bootstrapping, or frontend component input.

Audit relevance:
Context-specific encoding is required; HTML escaping alone may not protect script or URL contexts.

Follow-up:
- verify safe serialization, attribute encoding, URL scheme checks, and frontend consumers.

---

# 3. False-Positive Controls

Do not mark a C++ source as high-priority if:
- the value is fixed in trusted code,
- the value is only rendered by safe text APIs and no alternate raw/rich path is visible,
- the value never reaches HTML response construction, template rendering, frontend rendering, native HTML widgets, embedded browsers, markdown conversion, or browser-rendered output,
- stored content is trusted-only and cannot be attacker-influenced.

Use `Suspected source` or `Not enough evidence` if:
- wrapper/template behavior is hidden,
- sanitizer configuration is unavailable,
- frontend consumers are missing,
- stored writer paths are missing,
- raw vs escaped context is unclear.

---

# 4. Quick C++ Source Checklist

- Do HTTP/RPC/WebSocket/IPC/CGI inputs become HTML response data, template data, native UI text, or embedded browser content?
- Are stored comments, profiles, tickets, CMS content, markdown, or rich text later rendered?
- Are values named `html`, `safeHtml`, `trustedHtml`, `bodyHtml`, or `contentHtml` passed toward raw render paths?
- Are Qt/Wt/CEF/WebView/WebView2 and markdown paths using safe text or strict sanitization?
- Are backend JSON fields later rendered by frontend raw HTML or DOM sinks?
