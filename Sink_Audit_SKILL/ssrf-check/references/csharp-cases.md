# C# and .NET SSRF Cases

## Purpose

This file contains C#/.NET-specific SSRF patterns, candidate sink inventories, and audit cases.

Use it when the target application is implemented in C# or .NET, especially in:
- ASP.NET MVC, Web API, Razor Pages, minimal APIs, and ASP.NET Core
- SignalR hubs, gRPC services, WCF services, Azure Functions, and background workers
- HttpClient, WebRequest, RestSharp, Flurl, browser/rendering, XML/parser, and cloud SDK endpoint logic
- webhook tests, previews, importers, crawlers, metadata fetchers, and admin connectivity tools

This reference is guidance, not proof. Confirm attacker influence, final request target, sink behavior, and missing destination controls.

---

# 1. C# / .NET SSRF Control Points

## 1.1 Entry points
Look for:
- ASP.NET controllers and minimal APIs
- Razor Page handlers
- SignalR hub methods
- gRPC/WCF service methods
- Azure Functions triggers
- queue consumers and background workers
- admin, preview, import, and webhook handlers

## 1.2 Request sinks
Look for:
- HttpClient and WebRequest
- RestSharp and Flurl
- XML and document loaders
- browser/screenshot automation
- cloud SDK endpoint overrides

---

# 2. High-Coverage C# / .NET SSRF Candidate Inventory

## 2.1 ASP.NET, Web API, Razor Pages, and minimal API entry candidates
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
- `HttpRequest`
- `Request.Query`
- `MapGet`
- `MapPost`
- `PageModel`
- `OnGet`
- `OnPost`

## 2.2 RPC, function, worker, webhook, and admin entries
Search for:
- `Hub`
- `GrpcService`
- `ServerCallContext`
- `[OperationContract]`
- `[ServiceContract]`
- `HttpTrigger`
- `QueueTrigger`
- `ServiceBusTrigger`
- `TimerTrigger`
- `IHostedService`
- `BackgroundService`
- `ExecuteAsync`
- `Hangfire`
- `IConsumer`
- `webhook`
- `callback`
- `preview`
- `import`
- `metadata`
- `admin`

## 2.3 .NET outbound request sink candidates
Search for:
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

## 2.4 Parser, renderer, cloud SDK, and indirect fetch sink candidates
Search for:
- `XmlDocument.Load`
- `XDocument.Load`
- `XmlReader.Create`
- `XslCompiledTransform.Load`
- `Image.FromStream`
- `Playwright`
- `PuppeteerSharp`
- `Selenium WebDriver`
- `driver.Navigate().GoToUrl`
- `AmazonS3Config.ServiceURL`
- `AmazonS3Config.ProxyHost`
- `AWSOptions`
- `BlobClient`
- `ServiceBusClient`
- `endpoint`
- `BaseAddress`

## 2.5 Request target construction candidates
Search for:
- `string url`
- `string uri`
- `Uri`
- `UriBuilder`
- `HttpUtility.UrlDecode`
- `WebUtility.UrlDecode`
- `Convert.FromBase64String`
- `BaseAddress`
- `RequestUri`
- `callbackUrl`
- `webhookUrl`
- `imageUrl`
- `feedUrl`
- `endpoint`
- `host`
- `proxy`

## 2.6 Required-control candidates
Search near sinks for:
- `allowedHosts`
- `allowedSchemes`
- `Uri.Scheme`
- `Uri.Host`
- `Dns.GetHostAddresses`
- `IPAddress`
- `IsLoopback`
- private IP check
- link-local check
- `169.254.169.254`
- `localhost`
- `127.0.0.1`
- `::1`
- `AllowAutoRedirect = false`
- `MaxAutomaticRedirections`
- `SocketsHttpHandler`
- `ConnectCallback`
- timeout

## 2.7 C# / .NET graph search recipes
Useful combinations:

```text
[HttpPost] + HttpClient.SendAsync
[FromQuery] + WebRequest.Create
MapGet + WebClient.DownloadString
HttpTrigger + HttpClient.GetAsync
QueueTrigger + stored URL + HttpClient
AllowAutoRedirect + user URL
GrpcChannel.ForAddress + request value
AmazonS3Config.ServiceURL + user/stored endpoint
XmlDocument.Load + user URL
driver.Navigate().GoToUrl + preview URL
```

---

# 3. C# / .NET SSRF Anti-Patterns

### A1. Controller fetches attacker-controlled URL
```csharp
return await httpClient.GetStringAsync(url);
```

Why risky:
User-controlled URL may target internal or metadata services.

### A2. Redirect-following client with initial-only validation
```csharp
new HttpClientHandler { AllowAutoRedirect = true };
```

Why risky:
Redirect targets may change the final destination after validation.

### A3. Dynamic cloud SDK endpoint
```csharp
new AmazonS3Client(new AmazonS3Config { ServiceURL = endpoint });
```

Why risky:
Endpoint overrides can redirect SDK calls to attacker-selected hosts.

---

# 4. C# / .NET-Specific Audit Heuristics

Review HttpClient wrappers, IHttpClientFactory defaults, redirects, proxies, DNS/final IP checks, XML/browser fetchers, and SDK endpoint overrides.
