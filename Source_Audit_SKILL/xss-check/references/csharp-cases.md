# C# and .NET XSS Source Cases

## Purpose

This file contains C#/.NET-specific source point patterns and candidate search terms for XSS source discovery.

Use it when the target application is implemented in C# or .NET, especially in:
- ASP.NET MVC, Web API, Razor Pages, ASP.NET Core, and minimal APIs returning HTML
- Razor views, Tag Helpers, HtmlHelpers, Blazor components, and server-side rendering paths
- SignalR, gRPC, Azure Functions, queue consumers, hosted workers, emails, previews, dashboards, and reports that render user-controlled content
- markdown, rich text, CMS, admin, moderation, support, chat, and notification features

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify real source origin, propagation, trust boundary, downstream rendering relevance, and later context-specific controls.

---

# 1. High-Coverage C#/.NET XSS Source Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; keep it only when code evidence shows that the value can influence Razor/Blazor rendering, HTML responses, script data, attribute values, URL values, raw/trusted HTML wrappers, markdown/rich-text renderers, API-fed frontend rendering, emails, reports, or alternate render paths.

## 1.1 ASP.NET, Razor, and API entry candidates

Search for:
- `[ApiController]`
- `[Controller]`
- `[Route]`
- `[HttpGet]`
- `[HttpPost]`
- `[HttpPut]`
- `[HttpPatch]`
- `[HttpDelete]`
- `[FromBody]`
- `[FromForm]`
- `[FromQuery]`
- `[FromHeader]`
- `[FromRoute]`
- `ControllerBase`
- `Controller`
- `IActionResult`
- `ActionResult`
- `ViewResult`
- `PartialViewResult`
- `PageModel`
- `OnGet`
- `OnPost`
- `BindProperty`
- `HttpRequest`
- `Request.Query`
- `Request.Form`
- `Request.Cookies`
- `Request.Headers`
- `MapGet`
- `MapPost`
- `MapPut`
- `MapPatch`
- `MapDelete`

## 1.2 SignalR, function, worker, render, and admin entries

Search for:
- `Hub`
- `Hub<T>`
- `SendAsync`
- `Receive`
- `GrpcService`
- `ServerCallContext`
- `HttpTrigger`
- `QueueTrigger`
- `ServiceBusTrigger`
- `TimerTrigger`
- `IHostedService`
- `BackgroundService`
- `ExecuteAsync`
- `Hangfire`
- `IConsumer`
- `RazorViewToStringRenderer`
- `EmailTemplate`
- `Report`
- `Preview`
- `Admin`
- `Dashboard`
- `Comment`
- `Message`
- `Profile`
- moderation views

## 1.3 Reflected and stored content source candidates

Search for route, query, body, DTO, command, message, config, or model fields named:
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

## 1.4 Razor, Blazor, response, and API propagation candidates

Search for source values passed into:
- `ViewData`
- `ViewBag`
- `TempData`
- Razor model properties
- partial view models
- Tag Helper attributes
- `IHtmlContent`
- `HtmlString`
- `MvcHtmlString`
- `MarkupString`
- `RenderFragment`
- `builder.AddContent`
- `builder.AddMarkupContent`
- `ContentResult`
- `Content(`
- `Response.WriteAsync`
- `HttpResponse.WriteAsync`
- API response DTO fields
- SignalR payloads
- email template models
- report template models
- notification template models

## 1.5 Raw HTML, trusted wrapper, and context candidates

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
- `@Html.Raw`
- `Html.Raw`
- `new HtmlString`
- `MarkupString`
- `IHtmlContent`
- `WriteLiteral`
- `TagBuilder.InnerHtml`
- sanitizer output

## 1.6 Rich text, markdown, sanitizer, and cross-layer candidates

Search for source values near:
- `Markdig`
- `Markdown.ToHtml`
- `HtmlSanitizer`
- `Ganss.Xss`
- `AntiXssEncoder`
- `HtmlEncoder`
- `JavaScriptEncoder`
- `UrlEncoder`
- `JsonSerializer.Serialize`
- WYSIWYG editor content
- preview renderer
- final renderer
- admin renderer
- email/web preview renderer
- API field consumed by frontend
- Blazor component parameter
- script data bootstrap

## 1.7 Downstream rendering relevance mapping candidates

After finding a source candidate, trace toward:
- Razor view output
- `@Html.Raw`
- `Html.Raw`
- `HtmlString`
- `MarkupString`
- `builder.AddMarkupContent`
- `ContentResult`
- response writing
- Blazor rendering
- Tag Helpers
- markdown-to-HTML renderers
- sanitizer wrappers
- JSON API serializers
- frontend consumers
- admin/moderation views
- email/report renderers

## 1.8 C#/.NET graph search recipes

Useful combinations:

```text
[HttpGet]/[HttpPost] + [FromQuery]/[FromBody] message/content + ViewData/ViewBag/Model
Razor Page/PageModel + BindProperty/comment/body + Html.Raw/partial view
MapGet/MapPost + query/message + ContentResult/text/html
SignalR/QueueTrigger + stored message/notification + dashboard/email renderer
Markdown.ToHtml/Markdig + stored body + Html.Raw/MarkupString
HtmlSanitizer/Ganss.Xss + safeHtml/trustedHtml + raw Razor/Blazor render
builder.AddMarkupContent/MarkupString + API or stored content
JsonSerializer/hydrated state + API field html/bodyHtml + frontend consumer
```

---

# 2. C#/.NET Source Patterns

## C-S1. Request or DTO value becomes render model data

Example idea:
- query string, body DTO, route value, SignalR argument, gRPC message, or function payload becomes Razor model data, ViewData, ViewBag, Blazor parameter, or HTML response content.

Audit relevance:
The source may render safely or unsafely depending on Razor/Blazor context, raw helpers, and browser context.

Follow-up:
- trace into Razor views, Blazor components, HtmlHelpers, Tag Helpers, ContentResult, and frontend consumers.

## C-S2. Stored content display source

Example idea:
- comment, message, CMS body, profile field, ticket text, notification, email body, report label, or admin content is rendered later.

Audit relevance:
Stored content can affect other users, admins, reports, emails, and dashboards depending on display paths.

Follow-up:
- identify writer path, normal render path, admin path, email/report path, and preview/final consistency.

## C-S3. Raw HTML or trusted wrapper source

Example idea:
- values named `safeHtml`, `bodyHtml`, `MarkupString`, `HtmlString`, or sanitizer output are passed to Razor/Blazor raw rendering.

Audit relevance:
Trusted wrappers can represent either a safe transformation or untrusted content incorrectly marked raw.

Follow-up:
- inspect sanitizer configuration, trusted pipeline, and raw render destination.

## C-S4. Script, URL, attribute, or API-fed frontend source

Example idea:
- model or API fields become inline script data, attributes, `href`, `src`, JSON bootstrapping, SignalR payloads, or frontend component props.

Audit relevance:
Context-specific encoding is required; normal HTML encoding is not enough for every context.

Follow-up:
- verify safe serialization, attribute encoding, URL scheme controls, and frontend consumers.

---

# 3. False-Positive Controls

Do not mark a C#/.NET source as high-priority if:
- the value is fixed server-side,
- the value is only rendered by normal encoded Razor/Blazor text output and no alternate raw path is visible,
- the value never reaches Razor, Blazor, HTML response construction, frontend rendering, rich-content conversion, raw HTML helpers, or browser-rendered output,
- stored content is trusted-only and cannot be attacker-influenced.

Use `Suspected source` or `Not enough evidence` if:
- template behavior is hidden,
- sanitizer configuration is unavailable,
- frontend consumers are missing,
- stored writer paths are missing,
- raw vs escaped context is unclear.

---

# 4. Quick C#/.NET Source Checklist

- Are route, body, query, SignalR, Azure Function, queue, or worker values passed to Razor/Blazor render data?
- Are stored comments, profiles, tickets, CMS content, markdown, or rich text later rendered?
- Are values named `html`, `safeHtml`, `trustedHtml`, `bodyHtml`, or `MarkupString` passed toward raw render paths?
- Are preview, admin, email/web preview, report, export, and final display paths using the same source controls?
- Are backend JSON fields later rendered by frontend raw HTML or DOM sinks?
