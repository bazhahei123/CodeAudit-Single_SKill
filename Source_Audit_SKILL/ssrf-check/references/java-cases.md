# Java SSRF Source Cases

## Purpose

This file contains Java-specific source point patterns and audit cases for SSRF source discovery.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- `RestTemplate`
- `WebClient`
- `HttpURLConnection`
- Java `HttpClient`
- Apache HttpClient
- OkHttp
- Java backends exposing webhook tests, remote fetchers, previewers, importers, renderers, or generic HTTP helper services

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream outbound request behavior in the target code.

---

# 1. Java Source Discovery Points

Prioritize these source values and events:
- `@PathVariable`, `@RequestParam`, `@RequestBody`, headers, cookies, multipart metadata, and import DTO fields
- full URLs, callback URLs, webhook targets, preview URLs, image/file URLs, feed URLs, sitemap URLs, and remote import locations
- hostnames, IPs, ports, schemes, paths, query strings, base URLs, provider endpoints, tenant/integration endpoint records, localhost names, private-network addresses, link-local addresses, cloud metadata endpoints, and internal service aliases
- values passed into `URI.create(...)`, `new URL(...)`, `UriComponentsBuilder`, `RestTemplate`, `WebClient`, Java `HttpClient`, `HttpURLConnection`, Apache HttpClient, OkHttp, and outbound request wrappers
- redirect configuration, proxy settings, DNS-sensitive hostnames, parser-normalized URL fields, and final request-target values
- stored callback targets, webhook records, integration endpoints, queue payloads, job arguments, retry/replay data, and admin-configured remote targets

Source questions:
- Which source supplies full URL, host, IP, scheme, port, path, callback target, webhook endpoint, proxy, internal-service target, metadata endpoint, or redirect-relevant value?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value parsed, normalized, allowlisted, resolved, revalidated after redirects, stored, or passed through a wrapper before request use?
- Which SSRF context should be audited next: direct URL fetch, host recomposition, stored callback replay, redirect behavior, DNS/final-address checks, protocol handling, proxy behavior, renderer/importer, or network wrapper?

---

# 2. High-Coverage Java SSRF Source Candidate Inventory

Use these candidate lists to seed graph queries and text searches. Keep a candidate only when code shows request-target construction, URL parsing, URL recomposition, host/scheme/port/path selection, redirect behavior, DNS-sensitive target choice, proxy/client options, stored callback replay, renderer imports, or outbound request wrapper relevance.

## 2.1 HTTP, controller, and request entry candidates

Search for:
- `@RestController`
- `@Controller`
- `@RequestMapping`
- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@PatchMapping`
- `@DeleteMapping`
- `@RequestParam`
- `@PathVariable`
- `@RequestBody`
- `@RequestHeader`
- `@CookieValue`
- `@ModelAttribute`
- `MultipartFile`
- `HttpServletRequest`
- `ServletRequest`
- `request.getParameter`
- `request.getHeader`
- `@GraphQlController`
- `@QueryMapping`
- `@MutationMapping`
- `@SchemaMapping`
- JAX-RS `@Path`
- JAX-RS `@GET`
- JAX-RS `@POST`
- JAX-RS `@QueryParam`
- JAX-RS `@PathParam`

## 2.2 RPC, message, job, webhook, preview, and admin entries

Search for:
- `@GrpcService`
- `StreamObserver`
- `@MessageMapping`
- `@KafkaListener`
- `@RabbitListener`
- `@JmsListener`
- `@SqsListener`
- `@Scheduled`
- `@Async`
- Spring Batch `ItemReader`
- Spring Batch `ItemProcessor`
- `CommandLineRunner`
- `ApplicationRunner`
- `@EventListener`
- webhook controllers
- callback controllers
- preview controllers
- import controllers
- export controllers
- renderer controllers
- screenshot services
- crawler services
- metadata services
- admin connectivity tests
- replay handlers

## 2.3 Direct target and URL source candidates

Search for parameter, DTO, entity, map, or JSON fields named:
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

## 2.4 Partial destination, protocol, and client-option candidates

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
- `objectKey`
- `endpointOverride`
- `proxyHost`
- `proxyUrl`
- `proxy`
- `noProxy`
- `redirect`
- `followRedirects`
- `allowRedirects`
- `timeout`
- `ssl`
- `tls`
- `verifyHostname`

## 2.5 URL construction, parser, and normalization candidates

Search for source values near:
- `URI.create`
- `new URI`
- `new URL`
- `URLDecoder.decode`
- `Base64.getDecoder`
- `IDN.toASCII`
- `UriComponentsBuilder`
- `UriComponentsBuilder.fromUriString`
- `UriComponentsBuilder.fromHttpUrl`
- `UriBuilder`
- `HttpUrl.parse`
- `HttpUrl.Builder`
- `StringBuilder`
- `String.format`
- `MessageFormat.format`
- string concatenation around `http://`, `https://`, host, port, path, or query
- `InetAddress.getByName`
- `InetAddress.getAllByName`
- DNS resolver wrappers

## 2.6 Stored, callback, and indirect fetch source candidates

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
- S3 or storage service endpoint settings

## 2.7 Downstream SSRF relevance mapping candidates

After finding a source candidate, trace toward:
- `RestTemplate`
- `RestTemplate.getForObject`
- `RestTemplate.exchange`
- `WebClient`
- `WebClient.create`
- Java `HttpClient`
- `HttpRequest.newBuilder`
- `HttpClient.send`
- `HttpClient.sendAsync`
- `HttpURLConnection`
- `URL.openConnection`
- `URL.openStream`
- Apache HttpClient
- `CloseableHttpClient.execute`
- OkHttp `Request.Builder.url`
- `OkHttpClient`
- `AsyncHttpClient`
- `Jsoup.connect`
- Selenium or Playwright navigation
- cloud SDK endpoint override builders
- shared outbound request wrappers

## 2.8 Java graph search recipes

Useful combinations:

```text
@GetMapping/@PostMapping + @RequestParam/@RequestBody url/host + URI.create/new URL
@RestController + callbackUrl/webhookUrl/previewUrl + RestTemplate/WebClient/HttpClient
@RequestBody + endpoint/baseUrl/providerUrl + UriComponentsBuilder/client wrapper
@KafkaListener/@Scheduled + stored URL/callback + outbound wrapper
@QueryMapping/@MutationMapping + imageUrl/feedUrl/importUrl + fetch/preview/import helper
StringBuilder/String.format + request/stored host/path + HTTP client
HttpClient.Redirect.ALWAYS/followRedirects + request/stored URL
InetAddress/DNS resolver + host source + fetch wrapper
```

---

# 3. Java Source Patterns

## J-S1. Request-derived URL source
Example idea:
- request value such as `url`, `callback`, `target`, `imageUrl`, `feed`, `importUrl`, or `webhookUrl` becomes an outbound request target.

Audit relevance:
The source may affect the final server-side network destination.

Follow-up:
- trace into `RestTemplate`, `WebClient`, Java `HttpClient`, `HttpURLConnection`, Apache HttpClient, OkHttp, or shared request wrappers.

## J-S2. Host, path, scheme, or port recomposition source
Example idea:
- request or DTO fields provide host, path, port, scheme, tenant endpoint, provider name, or resource path that is combined into a URL.

Audit relevance:
Partial target sources can bypass checks when validation is applied before recomposition or to the wrong component.

Follow-up:
- verify strict mappings, canonical parsing, resolved-address checks, and final target validation.

## J-S3. Stored callback, webhook, or integration target source
Example idea:
- stored webhook URL, callback target, integration base URL, tenant endpoint, or provider URL is later fetched or called by a worker.

Audit relevance:
Stored targets create second-order SSRF source paths.

Follow-up:
- identify writer path and revalidation before every outbound request.

## J-S4. Redirect, DNS, and parser-sensitive source
Example idea:
- source values influence URL forms that are parsed, redirected, resolved, normalized, or fetched by different APIs.

Audit relevance:
The source may affect the final destination even if an initial string check appears safe.

Follow-up:
- inspect redirect-following behavior, DNS resolution, IP range checks, host canonicalization, userinfo handling, and parser differences.
- check whether client-controlled or weakly trusted values can name localhost, loopback, link-local, private-network, cloud metadata, or internal service destinations.

## J-S5. Protocol, proxy, and client-option source
Example idea:
- request, config, or stored value controls scheme, protocol, proxy, no-proxy behavior, redirect mode, TLS behavior, or client wrapper options.

Audit relevance:
Client options can expand reachable destinations or protocols.

Follow-up:
- verify allowed schemes, transport restrictions, proxy constraints, and wrapper defaults.

## J-S6. Indirect renderer/importer/fetcher source
Example idea:
- preview, screenshot, HTML/PDF/document rendering, image fetching, metadata crawling, or import logic accepts external resource references.

Audit relevance:
Indirect fetchers can create SSRF sources outside obvious HTTP client code.

Follow-up:
- trace renderer/importer library behavior and resource loading controls.

---

# 4. Case Templates

## Case J-S-SSRF-1: Direct URL source

Source focus:
Identify request or stored values that become outbound request targets.

Recommended follow-up:
Verify strict target allowlists, scheme restrictions, redirect policy, and resolved-address checks.

## Case J-S-SSRF-2: Callback or webhook target source

Source focus:
Identify callback, webhook, or integration endpoints that are submitted, stored, tested, retried, or replayed.

Recommended follow-up:
Trace writer and fetch paths and verify revalidation before use.

## Case J-S-SSRF-3: URL component source

Source focus:
Identify host, path, scheme, port, or base URL values used to construct a final request target.

Recommended follow-up:
Verify validation applies to the final canonical target, not only the original component.

## Case J-S-SSRF-4: Indirect fetch source

Source focus:
Identify preview/import/render/crawler source values that cause library-managed outbound fetches.

Recommended follow-up:
Inspect library configuration and resource loading restrictions.

---

# 5. Java-Specific Audit Heuristics

## 5.1 Spring route and DTO source heuristics
Pay attention to:
- `@RequestParam`
- `@PathVariable`
- `@RequestBody` DTO fields
- `@RequestHeader`
- multipart metadata
- GraphQL resolver arguments
- webhook, preview, import, render, and admin route parameters

## 5.2 URL and URI construction source heuristics
Pay attention to:
- `URI.create(...)`
- `new URL(...)`
- `UriComponentsBuilder`
- string-built URLs
- host/path/base URL recomposition
- callback and preview target assembly
- provider or tenant endpoint mapping

## 5.3 Client API source heuristics
Pay attention to:
- `RestTemplate`
- `WebClient`
- Java `HttpClient`
- `HttpURLConnection`
- Apache HttpClient
- OkHttp
- shared outbound request wrappers

## 5.4 Redirect, DNS, and proxy source heuristics
Pay attention to:
- automatic redirect configuration
- custom redirect handling
- proxy/no-proxy behavior
- DNS resolution and IP range checks
- wrapper defaults hidden in shared HTTP clients

## 5.5 Indirect and stored source heuristics
Pay attention to:
- OpenGraph and metadata preview
- image/file remote loaders
- webhook test features
- import/render jobs
- queue workers using stored URLs
- saved integration or tenant endpoints

---

# 6. False-Positive Controls

Do not mark a Java source as high-priority if:
- the value is selected from a strict allowlist of safe fixed endpoints,
- the final URL is built entirely from trusted server-side constants,
- the value never reaches request target construction, URL parsing, remote fetch wrappers, or outbound clients,
- the stored target is trusted-only and cannot be attacker-influenced,
- the source only affects display labels, logs, or non-network metadata.

Use `Suspected source` or `Not enough evidence` if:
- wrapper behavior is hidden,
- redirect behavior is unclear,
- DNS/final-address checks are not visible,
- stored target writer paths are missing,
- URL validation may exist elsewhere.

---

# 7. What Good Evidence Looks Like

Good Java source evidence includes:
- route/controller/worker/admin/import/render entry point,
- source annotation or API such as request param, DTO field, queue receive, config record, or stored callback record,
- propagation such as URL construction, URI parsing, host/path recomposition, storage, redirect option setting, proxy option construction, or wrapper call,
- `RestTemplate`, `WebClient`, Java `HttpClient`, `HttpURLConnection`, Apache HttpClient, OkHttp, renderer/importer, or network wrapper receiving the value,
- SSRF context when visible.

Good source evidence answers:
1. Which Java entry point receives the outbound-request-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which direct fetch, URL construction, stored callback, redirect, DNS, protocol, proxy, wrapper, renderer, or worker behavior should be audited next?
4. Is the source used for full URL, host, IP, scheme, port, path, redirect target, proxy, metadata endpoint, internal-service target, stored endpoint, or indirect resource reference?

---

# 8. Quick Java Source Checklist

- Are request values used as full URLs, callback targets, hosts, ports, paths, schemes, or remote resource references?
- Can request or stored values name localhost, loopback, link-local, private-network, cloud metadata, or internal service targets?
- Are stored webhook, callback, integration, or tenant endpoints later fetched?
- Are redirects, DNS names, parser output, or final resolved addresses influenced?
- Are proxy, protocol, or client options dynamic?
- Are preview/import/render/crawler features fetching externally supplied resources?
- Are background jobs and retries revalidating stored targets before request use?
