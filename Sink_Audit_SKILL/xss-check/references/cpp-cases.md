# C++ XSS Cases

## Purpose

This file contains C++-specific XSS and unsafe HTML rendering patterns, candidate sink inventories, and audit cases.

Use it when the target application includes C++ or native service code that emits HTML, renders web UI, hosts embedded browsers, or generates previews, especially in:
- HTTP servers, CGI/FastCGI handlers, RPC/IPC handlers, and native admin dashboards
- Crow, Drogon, oatpp, cpp-httplib, Pistache, Wt, CppCMS, Qt, CEF, WebView, and template engines
- server-side HTML generation, native WebView rendering, markdown/rich-text preview, reports, and email HTML generation

This reference is guidance, not proof. Confirm attacker influence, rendering context, sink behavior, and missing controls.

---

# 1. C++ XSS Control Points

## 1.1 Entry and render points
Look for:
- HTTP route handlers
- CGI/FastCGI handlers
- RPC or IPC methods feeding HTML generation
- native UI screens that host WebView/CEF/Qt WebEngine
- admin, report, preview, support, chat, CMS, and notification views

## 1.2 Sink points
Look for:
- response body writes with `text/html`
- template rendering with raw variables
- Wt/Qt/CEF/WebView HTML APIs
- markdown/rich-text conversion
- manual HTML string concatenation

---

# 2. High-Coverage C++ XSS Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows.

## 2.1 HTTP, RPC, WebSocket, and IPC entry candidates
Search for:
- `CROW_ROUTE`
- `crow::SimpleApp`
- `app.route_dynamic`
- `DROGON_BEGIN_NAMESPACE`
- `METHOD_LIST_BEGIN`
- `ADD_METHOD_TO`
- `HttpController`
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
- `onMessage`
- `WebSocket`
- `DBus`
- `Binder`
- `onTransact`

## 2.2 HTML response and template sink candidates
Search for:
- `text/html`
- `setContentType`
- `Content-Type: text/html`
- `response.write`
- `res.write`
- `res.body`
- `HttpResponse::newHttpResponse`
- `setBody`
- `std::string html`
- `std::stringstream html`
- `fmt::format`
- `QString::arg`
- `inja`
- `inja::Environment`
- `mustache`
- `mstch`
- `ctemplate`
- `inja::render`
- `Template.render`
- `raw`
- `safeHtml`

## 2.3 Native WebView/UI HTML sink candidates
Search for:
- `Wt::WText`
- `Wt::TextFormat::XHTML`
- `Wt::TextFormat::XHTMLUnsafeText`
- `Wt::WTemplate`
- `QTextDocument::setHtml`
- `QTextBrowser::setHtml`
- `QLabel::setText`
- `Qt::RichText`
- `QWebEngineView::setHtml`
- `QWebEngineView::load`
- `QWebEnginePage::runJavaScript`
- `QWebView::setHtml`
- `CEF`
- `CefBrowser`
- `CefFrame::LoadString`
- `WebView2`
- `NavigateToString`

## 2.4 Markdown, rich text, and sanitizer candidates
Search for:
- `markdown`
- `cmark`
- `md4c`
- `Discount`
- `Hoedown`
- `CommonMark`
- `WYSIWYG`
- `bodyHtml`
- `contentHtml`
- `sanitize`
- `escapeHtml`
- `htmlEscape`
- `QString::toHtmlEscaped`
- `Wt::Utils::htmlEncode`
- `Poco::Net::HTMLForm`
- `href`
- `src`
- `onclick`
- `script`

## 2.5 Required-control candidates
Search near sinks for:
- HTML escape
- `escapeHtml`
- `htmlEscape`
- `QString::toHtmlEscaped`
- `Wt::Utils::htmlEncode`
- `Wt::TextFormat::Plain`
- no `XHTMLUnsafeText`
- no raw template output
- sanitizer
- allowed tags
- allowed attributes
- URL scheme allowlist
- safe JSON serialization
- `Content-Security-Policy`
- preview/final render consistency

## 2.6 C++ graph search recipes
Useful combinations:

```text
CROW_ROUTE + text/html + request parameter
svr.Get + std::stringstream html
Drogon controller + setBody + user content
inja render + raw variable
Wt::WText + XHTMLUnsafeText
QTextDocument::setHtml + user content
QWebEngineView::setHtml + IPC/request value
CefFrame::LoadString + user HTML
markdown renderer + response text/html
```

---

# 3. C++ XSS Anti-Patterns

### A1. HTTP handler concatenates HTML
```cpp
std::string html = "<div>" + name + "</div>";
res.setBody(html);
```

Why risky:
User-controlled content is inserted into HTML without escaping.

### A2. Qt rich text from external input
```cpp
label->setText(userContent);
label->setTextFormat(Qt::RichText);
```

Why risky:
Qt rich text mode interprets markup-like content.

### A3. Embedded browser loads attacker-controlled HTML
```cpp
webView->setHtml(userHtml);
```

Why risky:
The embedded browser interprets the value as active HTML.

---

# 4. C++-Specific Audit Heuristics

Review native HTTP response writers, template raw output, Wt text formats, Qt rich text and WebEngine APIs, CEF/WebView HTML loading, markdown/rich-text conversion, and context-specific escaping.
