# C# and .NET SSRF Source Cases

## Purpose

This file contains C#/.NET-specific source point patterns and candidate search terms for SSRF source discovery.

Use it when the target application is implemented in C# or .NET, especially in:
- ASP.NET MVC, Web API, Razor Pages, minimal APIs, and ASP.NET Core
- SignalR hubs, gRPC services, WCF services, Azure Functions, queue consumers, and hosted workers
- HttpClient, WebRequest, RestSharp, Flurl, Refit, browser/rendering, XML/parser, and cloud SDK endpoint logic
- webhook tests, previews, importers, crawlers, metadata fetchers, admin connectivity tools, and tenant endpoint settings

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify real source origin, propagation, trust boundary, downstream SSRF relevance, and later target controls.

---

# 1. High-Coverage C#/.NET SSRF Source Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; keep it only when code evidence shows that the value can influence request-target construction, URL parsing, URL recomposition, host/scheme/port/path selection, redirect behavior, DNS-sensitive target choice, proxy/client options, stored callback replay, renderer imports, cloud SDK endpoint overrides, or outbound request wrapper behavior.

## 1.1 ASP.NET, Web API, Razor Pages, and minimal API entry candidates

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
- `MapGroup`
- `PageModel`
- `OnGet`
- `OnPost`
- `BindProperty`

## 1.2 RPC, function, worker, webhook, preview, and admin entries

Search for:
- `Hub`
- `Hub<T>`
- `GrpcService`
- `ServerCallContext`
- `[OperationContract]`
- `[ServiceContract]`
- `[WebMethod]`
- `HttpTrigger`
- `QueueTrigger`
- `ServiceBusTrigger`
- `TimerTrigger`
- `IHostedService`
- `BackgroundService`
- `ExecuteAsync`
- `Hangfire`
- `IConsumer`
- `Handle`
- `Consume`
- `MediatR`
- `IRequestHandler`
- webhook handlers
- callback handlers
- preview handlers
- import handlers
- metadata handlers
- admin connectivity tests
- replay handlers

## 1.3 Direct target and URL source candidates

Search for route, query, body, DTO, command, message, config, or model fields named:
- `url`
- `uri`
- `target`
- `targetUrl`
- `requestUrl`
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
- `openGraphUrl`
- `endpoint`
- `baseUrl`
- `serviceUrl`
- `providerUrl`
- `tenantUrl`
- `integrationUrl`

## 1.4 Partial destination, protocol, and client-option candidates

Search for:
- `host`
- `hostname`
- `domain`
- `ip`
- `address`
- `serviceName`
- `scheme`
- `protocol`
- `port`
- `path`
- `route`
- `query`
- `resource`
- `endpointOverride`
- `proxyHost`
- `proxyUrl`
- `proxy`
- `noProxy`
- `AllowAutoRedirect`
- `MaxAutomaticRedirections`
- `BaseAddress`
- `RequestUri`
- `timeout`
- `ssl`
- `tls`
- `DangerousAcceptAnyServerCertificateValidator`

## 1.5 URL construction, parser, and normalization candidates

Search for source values near:
- `new Uri`
- `Uri.TryCreate`
- `UriBuilder`
- `HttpUtility.UrlDecode`
- `WebUtility.UrlDecode`
- `Convert.FromBase64String`
- `Uri.EscapeDataString`
- `Uri.UnescapeDataString`
- `Uri.Host`
- `Uri.Scheme`
- `Uri.Port`
- `Dns.GetHostAddresses`
- `IPAddress.Parse`
- `IPAddress.TryParse`
- `IdnMapping`
- `string.Format`
- `$"{`
- `StringBuilder`
- string concatenation around URL pieces

## 1.6 Stored, callback, and indirect fetch source candidates

Search for:
- webhook registration records
- callback target records
- integration endpoint records
- tenant endpoint settings
- provider endpoint settings
- saved crawler targets
- URL preview records
- import job URLs
- retry or replay payloads
- queue payload URLs
- remote image or avatar URLs
- OpenGraph metadata URLs
- feed or sitemap URLs
- PDF/HTML/Markdown renderer inputs
- browser automation preview URLs
- cloud SDK endpoint overrides
- storage service endpoint settings

## 1.7 Downstream SSRF relevance mapping candidates

After finding a source candidate, trace toward:
- `HttpClient.GetAsync`
- `HttpClient.PostAsync`
- `HttpClient.SendAsync`
- `HttpRequestMessage`
- `IHttpClientFactory`
- `WebClient.DownloadString`
- `WebClient.OpenRead`
- `WebRequest.Create`
- `HttpWebRequest`
- `RestSharp.RestClient`
- `RestRequest`
- `Flurl`
- `Refit`
- `GrpcChannel.ForAddress`
- `TcpClient.Connect`
- `Socket.Connect`
- `XmlDocument.Load`
- `XDocument.Load`
- `XmlReader.Create`
- `Image.FromStream`
- Playwright/Puppeteer/Selenium navigation
- `AmazonS3Config.ServiceURL`
- `ServiceURL`
- `Endpoint`
- `BlobClient`
- shared outbound request wrappers

## 1.8 C#/.NET graph search recipes

Useful combinations:

```text
[HttpGet]/[HttpPost] + [FromQuery]/[FromBody] url/host + new Uri/UriBuilder
MapGet/MapPost + callbackUrl/webhookUrl/previewUrl + HttpClient/WebRequest
HttpTrigger/QueueTrigger + stored URL/callback + IHttpClientFactory/HttpClient
SignalR/gRPC/WCF method + endpoint/baseUrl/providerUrl + client wrapper
StringBuilder/string.Format/$ interpolation + request/stored host/path + outbound request
AllowAutoRedirect + request/stored URL
Dns.GetHostAddresses/IPAddress + host source + fetch wrapper
XmlDocument.Load/driver.Navigate().GoToUrl + preview/import source
AmazonS3Config.ServiceURL/GrpcChannel.ForAddress + request/stored endpoint
```

---

# 2. C#/.NET Source Patterns

## C-S1. Request or DTO value becomes request target

Example idea:
- query string, body DTO, route value, SignalR argument, gRPC message, or function payload becomes a URL, host, endpoint, callback, preview, or import target.

Audit relevance:
The source may affect the final server-side network destination.

Follow-up:
- trace into HttpClient, WebRequest, RestSharp, Flurl, Refit, gRPC channel builders, raw sockets, cloud SDKs, and shared wrappers.

## C-S2. Stored callback, webhook, or integration target source

Example idea:
- stored webhook URL, callback target, tenant endpoint, integration base URL, or provider endpoint is later fetched by a worker or retry path.

Audit relevance:
Stored targets create second-order SSRF source paths.

Follow-up:
- identify writer path, storage path, retry path, and revalidation before every fetch.

## C-S3. Redirect, DNS, proxy, or client-option source

Example idea:
- input controls redirect behavior, proxy host, base address, DNS-sensitive host, endpoint override, or TLS/client settings.

Audit relevance:
Client options can alter final destination, network routing, or reachable protocol surface.

Follow-up:
- verify final resolved-address checks, redirect revalidation, proxy restrictions, and trusted config origins.

## C-S4. Indirect renderer, parser, or cloud SDK source

Example idea:
- preview, XML, document, image, browser automation, or cloud SDK endpoint configuration receives externally supplied targets.

Audit relevance:
These paths can hide network fetches outside ordinary HttpClient calls.

Follow-up:
- inspect parser/resource loading behavior and SDK endpoint override handling.

---

# 3. False-Positive Controls

Do not mark a C#/.NET source as high-priority if:
- the value is fixed server-side or selected from a strict endpoint allowlist,
- the value affects only display, logs, analytics, or non-network metadata,
- the value never reaches URL construction, request clients, renderer/parser fetch behavior, cloud SDK endpoint builders, or network wrappers,
- stored endpoint values are trusted-only and cannot be attacker-influenced.

Use `Suspected source` or `Not enough evidence` if:
- wrapper behavior is hidden,
- redirect behavior, DNS checks, or proxy behavior is unclear,
- `IHttpClientFactory` defaults are unavailable,
- stored target writer paths are missing.

---

# 4. Quick C#/.NET Source Checklist

- Are route, body, query, SignalR, gRPC, Azure Function, queue, or worker values used as URLs or URL components?
- Do request or stored values influence `BaseAddress`, `RequestUri`, host, scheme, port, proxy, redirect, endpoint override, or cloud SDK service URL?
- Are XML/browser/document/image/parser fetches fed by externally supplied targets?
- Are saved callbacks, webhooks, tenant endpoints, integration endpoints, or retry payloads revalidated before fetch?
- Is the source only mapped to a fixed trusted endpoint, or can it alter the final request target?
