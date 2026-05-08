# C# and .NET XSS Cases

## Purpose

This file contains C#/.NET-specific XSS patterns, candidate sink inventories, and audit cases.

Use it when the target application is implemented in C# or .NET, especially in:
- ASP.NET MVC, Web API, Razor Pages, ASP.NET Core, and minimal APIs returning HTML
- Razor views, Tag Helpers, HtmlHelpers, Blazor components, and server-side rendering paths
- SignalR, gRPC, Azure Functions, background jobs, emails, previews, dashboards, and reports that render user-controlled content
- markdown, rich text, CMS, admin, moderation, support, chat, and notification features

This reference is guidance, not proof. Confirm attacker influence, rendering context, sink behavior, and missing context-appropriate controls.

---

# 1. C# / .NET XSS Control Points

## 1.1 Entry and render points
Look for:
- MVC controllers
- Razor Pages
- minimal API endpoints returning HTML
- Blazor components
- SignalR hubs
- Azure Functions returning HTML
- background jobs creating HTML emails or reports
- admin/moderation/support dashboards

## 1.2 Sink points
Look for:
- Razor raw rendering
- manual `ContentResult` / `HtmlString`
- response writing
- Blazor `MarkupString`
- markdown/rich-text conversion
- JavaScript or attribute context rendering

---

# 2. High-Coverage C# / .NET XSS Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows.

## 2.1 ASP.NET, Razor, and API entry candidates
Search for:
- `[Controller]`
- `[ApiController]`
- `[Route]`
- `[HttpGet]`
- `[HttpPost]`
- `[HttpPut]`
- `[HttpPatch]`
- `[HttpDelete]`
- `[FromBody]`
- `[FromForm]`
- `[FromQuery]`
- `[FromRoute]`
- `Controller`
- `ControllerBase`
- `IActionResult`
- `ActionResult`
- `ViewResult`
- `PartialViewResult`
- `PageModel`
- `OnGet`
- `OnPost`
- `MapGet`
- `MapPost`
- `HttpRequest`
- `Request.Query`
- `Request.Form`

## 2.2 SignalR, function, worker, and rendering entries
Search for:
- `Hub`
- `Hub<T>`
- `SendAsync`
- `Receive`
- `HttpTrigger`
- `QueueTrigger`
- `ServiceBusTrigger`
- `IHostedService`
- `BackgroundService`
- `ExecuteAsync`
- `RazorViewToStringRenderer`
- `EmailTemplate`
- `Report`
- `Preview`
- `Admin`
- `Dashboard`
- `Comment`
- `Message`
- `Profile`

## 2.3 Razor, MVC, and manual HTML sink candidates
Search for:
- `@Html.Raw`
- `Html.Raw`
- `IHtmlContent`
- `HtmlString`
- `new HtmlString`
- `MvcHtmlString`
- `TagBuilder.InnerHtml`
- `TagBuilder.Attributes`
- `Content(`
- `ContentResult`
- `Response.WriteAsync`
- `HttpResponse.WriteAsync`
- `WriteLiteral`
- `WriteTo`
- `RazorPage.WriteLiteral`
- `Partial(`
- `RenderPartial`
- `ViewData`
- `ViewBag`
- `TempData`
- `Raw`

## 2.4 Blazor, markdown, sanitizer, and script/URL candidates
Search for:
- `MarkupString`
- `RenderFragment`
- `builder.AddMarkupContent`
- `builder.AddContent`
- `@((MarkupString)`
- `JSRuntime.InvokeAsync`
- `IJSRuntime`
- `Markdig`
- `Markdown.ToHtml`
- `HtmlSanitizer`
- `Ganss.Xss`
- `AntiXssEncoder`
- `HtmlEncoder`
- `JavaScriptEncoder`
- `UrlEncoder`
- `System.Text.Encodings.Web`
- `href`
- `src`
- `onclick`
- `script`

## 2.5 Required-control candidates
Search near sinks for:
- Razor default encoding
- no `Html.Raw`
- no `MarkupString`
- `HtmlEncoder.Encode`
- `JavaScriptEncoder.Encode`
- `UrlEncoder.Encode`
- `HttpUtility.HtmlEncode`
- `AntiXssEncoder.HtmlEncode`
- `JsonSerializer.Serialize`
- safe JSON serialization
- `HtmlSanitizer.Sanitize`
- allowed tags
- allowed attributes
- URL scheme allowlist
- `Content-Security-Policy`
- preview/final render consistency

## 2.6 C# / .NET graph search recipes
Useful combinations:

```text
[HttpGet] + Html.Raw
[FromQuery] + ContentResult
Razor Page + @Html.Raw
ViewBag + Html.Raw
MarkupString + user content
builder.AddMarkupContent + API data
Markdig Markdown.ToHtml + Html.Raw
SignalR message + raw render
admin dashboard + user content + HtmlString
```

---

# 3. C# / .NET XSS Anti-Patterns

### A1. Razor raw rendering of user content
```csharp
@Html.Raw(Model.CommentBody)
```

Why risky:
Raw Razor rendering bypasses normal HTML encoding.

### A2. Blazor MarkupString from user data
```csharp
builder.AddContent(0, (MarkupString)comment.Body);
```

Why risky:
`MarkupString` treats content as markup instead of text.

### A3. Manual HTML response
```csharp
return Content("<div>" + message + "</div>", "text/html");
```

Why risky:
Manual HTML construction can reflect attacker-controlled content without encoding.

---

# 4. C# / .NET-Specific Audit Heuristics

Review Razor raw helpers, Blazor markup sinks, manual HTML responses, JavaScript/URL contexts, markdown/rich-text pipelines, and admin/email/report rendering consistency.
