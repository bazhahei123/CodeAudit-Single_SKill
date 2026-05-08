# PHP SSRF Source Cases

## Purpose

This file contains PHP-specific source point patterns and audit cases for SSRF source discovery.

Use it when the target application is primarily implemented in PHP, especially in:
- Laravel
- Symfony
- raw PHP applications
- cURL
- `file_get_contents` with remote URLs
- Guzzle
- framework HTTP clients
- PHP backends exposing webhook tests, remote fetchers, previewers, importers, renderers, or generic HTTP helper services

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream outbound request behavior in the target code.

---

# 1. PHP Source Discovery Points

Prioritize these source values and events:
- `$_GET`, `$_POST`, `$_REQUEST`, route parameters, request input, headers, cookies, uploaded metadata, and import rows
- full URLs, callback URLs, webhook targets, preview URLs, image/file URLs, feed URLs, sitemap URLs, and remote import locations
- hostnames, IPs, ports, schemes, paths, query strings, base URLs, provider endpoints, tenant/integration endpoint records, localhost names, private-network addresses, link-local addresses, cloud metadata endpoints, and internal service aliases
- values passed into cURL, `file_get_contents`, stream wrappers, Guzzle, Laravel HTTP client, Symfony HTTP client, framework clients, helper wrappers, previewers, crawlers, importers, and renderers
- redirect options such as `CURLOPT_FOLLOWLOCATION`, protocol options, proxy settings, DNS-sensitive hostnames, parser-normalized URL fields, and final request-target values
- stored callback targets, webhook records, integration endpoints, queue/job payloads, cron/admin task definitions, retry/replay data, and admin-configured remote targets

Source questions:
- Which source supplies full URL, host, IP, scheme, port, path, callback target, webhook endpoint, proxy, internal-service target, metadata endpoint, or redirect-relevant value?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value parsed, normalized, allowlisted, resolved, revalidated after redirects, stored, or passed through a wrapper before request use?
- Which SSRF context should be audited next: direct URL fetch, host recomposition, stored callback replay, redirect behavior, DNS/final-address checks, protocol handling, proxy behavior, renderer/importer, or network wrapper?

---

# 2. High-Coverage PHP SSRF Source Candidate Inventory

Use these candidate lists to seed graph queries and text searches. Keep a candidate only when code shows request-target construction, URL parsing, URL recomposition, host/scheme/port/path selection, redirect behavior, DNS-sensitive target choice, proxy/client options, stored callback replay, renderer imports, or outbound request wrapper relevance.

## 2.1 Web, framework, and request entry candidates

Search for:
- Laravel `Route::get`
- Laravel `Route::post`
- Laravel `Route::put`
- Laravel `Route::patch`
- Laravel `Route::delete`
- Laravel controller methods
- `$request->input`
- `$request->query`
- `$request->post`
- `$request->all`
- `$request->only`
- `$request->validated`
- `$request->header`
- route parameters
- Symfony `#[Route]`
- Symfony `Request`
- `$request->query->get`
- `$request->request->get`
- `$request->headers->get`
- ThinkPHP controllers
- Yii controllers
- CodeIgniter controllers
- WordPress `add_action`
- WordPress REST routes
- raw `$_GET`
- `$_POST`
- `$_REQUEST`
- `$_COOKIE`
- `$_SERVER`
- uploaded file metadata

## 2.2 Queue, command, webhook, preview, import, and admin entries

Search for:
- Laravel jobs
- Laravel listeners
- Laravel events
- Laravel queued jobs
- Laravel console commands
- Symfony commands
- Symfony Messenger handlers
- cron scripts
- webhook controllers
- callback controllers
- preview controllers
- import controllers
- export controllers
- renderer jobs
- screenshot jobs
- crawler jobs
- metadata jobs
- admin connectivity tests
- replay handlers
- ETL/sync tasks
- legacy admin scripts

## 2.3 Direct target and URL source candidates

Search for request, DTO, array, model, form, job, or config fields named:
- `url`
- `uri`
- `target`
- `target_url`
- `request_url`
- `remote_url`
- `external_url`
- `callback`
- `callback_url`
- `webhook`
- `webhook_url`
- `redirect_url`
- `return_url`
- `preview_url`
- `image_url`
- `avatar_url`
- `file_url`
- `download_url`
- `import_url`
- `feed_url`
- `sitemap_url`
- `metadata_url`
- `open_graph_url`
- `endpoint`
- `base_url`
- `service_url`
- `provider_url`
- `tenant_url`
- `integration_url`

## 2.4 Partial destination, protocol, and client-option candidates

Search for:
- `host`
- `hostname`
- `domain`
- `ip`
- `address`
- `service_name`
- `scheme`
- `protocol`
- `port`
- `path`
- `route`
- `query`
- `resource`
- `object_key`
- `endpoint_override`
- `proxy`
- `proxy_host`
- `proxy_url`
- `no_proxy`
- `follow_location`
- `allow_redirects`
- `timeout`
- `ssl`
- `verify`
- `stream_context`

## 2.5 URL construction, parser, and normalization candidates

Search for source values near:
- `parse_url`
- `http_build_query`
- `urldecode`
- `rawurldecode`
- `base64_decode`
- `idn_to_ascii`
- `filter_var`
- `GuzzleHttp\\Psr7\\Uri`
- `League\\Uri`
- `sprintf`
- `vsprintf`
- `implode`
- string interpolation around `http://` or `https://`
- concatenation around URL pieces
- `gethostbyname`
- `dns_get_record`
- `inet_pton`
- custom resolver helpers

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
- storage service endpoint settings

## 2.7 Downstream SSRF relevance mapping candidates

After finding a source candidate, trace toward:
- `curl_init`
- `curl_setopt`
- `curl_exec`
- `CURLOPT_URL`
- `file_get_contents`
- `fopen`
- `readfile`
- `copy`
- `get_headers`
- `stream_context_create`
- `GuzzleHttp\\Client`
- `Client::request`
- Laravel `Http::get`
- Laravel `Http::post`
- Symfony HTTP client
- WordPress HTTP API
- `wp_remote_get`
- `wp_remote_post`
- `SoapClient`
- XML/HTML parsers and importers
- browser/screenshot wrappers
- shared outbound request wrappers

## 2.8 PHP graph search recipes

Useful combinations:

```text
Route::get/Route::post + $request->input url/host + parse_url/http_build_query
$request->query/$_GET + callback_url/webhook_url + curl/Guzzle/Http client
Symfony #[Route] + endpoint/base_url/provider_url + HTTP client wrapper
Laravel job/command + stored URL/callback + Http::get/Guzzle/cURL
sprintf/implode/concat + request/stored host/path + outbound wrapper
CURLOPT_FOLLOWLOCATION/allow_redirects + request/stored URL
gethostbyname/dns_get_record/inet_pton + host source + fetch wrapper
file_get_contents/fopen/copy + preview/import/image/feed URL
```

---

# 3. PHP Source Patterns

## H-S1. Request-derived URL source
Example idea:
- request value such as `url`, `callback`, `target`, `image_url`, `feed`, `import_url`, or `webhook_url` becomes an outbound request target.

Audit relevance:
The source may affect the final server-side network destination.

Follow-up:
- trace into cURL, `file_get_contents`, stream wrappers, Guzzle, framework HTTP clients, or shared request wrappers.

## H-S2. Host, path, scheme, or port recomposition source
Example idea:
- request fields provide host, path, port, scheme, tenant endpoint, provider name, or resource path that is combined into a URL.

Audit relevance:
Partial target sources can bypass checks when validation is applied before recomposition or to the wrong component.

Follow-up:
- verify strict mappings, canonical parsing, resolved-address checks, and final target validation.

## H-S3. Stored callback, webhook, or integration target source
Example idea:
- stored webhook URL, callback target, integration base URL, tenant endpoint, or provider URL is later fetched or called by a worker.

Audit relevance:
Stored targets create second-order SSRF source paths.

Follow-up:
- identify writer path and revalidation before every outbound request.

## H-S4. Redirect, DNS, and parser-sensitive source
Example idea:
- source values influence URL forms that are parsed, redirected, resolved, normalized, or fetched by different APIs.

Audit relevance:
The source may affect the final destination even if an initial string check appears safe.

Follow-up:
- inspect redirect-following behavior, DNS resolution, IP range checks, host canonicalization, userinfo handling, and parser differences.
- check whether client-controlled or weakly trusted values can name localhost, loopback, link-local, private-network, cloud metadata, or internal service destinations.

## H-S5. Protocol, proxy, and client-option source
Example idea:
- request, config, or stored value controls scheme, protocol, proxy, no-proxy behavior, redirect behavior, TLS behavior, stream wrapper behavior, or client wrapper options.

Audit relevance:
Client options can expand reachable destinations or protocols.

Follow-up:
- verify allowed schemes, transport restrictions, proxy constraints, stream wrapper restrictions, and wrapper defaults.

## H-S6. Indirect renderer/importer/fetcher source
Example idea:
- preview, screenshot, HTML/PDF/document rendering, image fetching, metadata crawling, or import logic accepts external resource references.

Audit relevance:
Indirect fetchers can create SSRF sources outside obvious HTTP client code.

Follow-up:
- trace renderer/importer library behavior and resource loading controls.

---

# 4. Case Templates

## Case H-S-SSRF-1: Direct URL source

Source focus:
Identify request or stored values that become outbound request targets.

Recommended follow-up:
Verify strict target allowlists, scheme restrictions, redirect policy, and resolved-address checks.

## Case H-S-SSRF-2: Callback or webhook target source

Source focus:
Identify callback, webhook, or integration endpoints that are submitted, stored, tested, retried, or replayed.

Recommended follow-up:
Trace writer and fetch paths and verify revalidation before use.

## Case H-S-SSRF-3: URL component source

Source focus:
Identify host, path, scheme, port, or base URL values used to construct a final request target.

Recommended follow-up:
Verify validation applies to the final canonical target, not only the original component.

## Case H-S-SSRF-4: Indirect fetch source

Source focus:
Identify preview/import/render/crawler source values that cause library-managed outbound fetches.

Recommended follow-up:
Inspect library configuration and resource loading restrictions.

---

# 5. PHP-Specific Audit Heuristics

## 5.1 Request and framework source heuristics
Pay attention to:
- `$_GET`, `$_POST`, and `$_REQUEST`
- `$request->input(...)`
- route parameters
- validation result arrays later used in HTTP helpers
- uploaded metadata and import rows
- webhook, preview, import, render, and admin route parameters

## 5.2 URL assembly source heuristics
Pay attention to:
- string-built URLs
- `parse_url(...)` parsing and recomposition
- host/path/base URL composition
- callback URL generation
- preview/import target construction
- helper methods named `fetch`, `download`, `preview`, `verify`, `crawl`, or `import`

## 5.3 Client API source heuristics
Pay attention to:
- cURL
- `file_get_contents` with URLs
- stream wrappers
- Guzzle
- Laravel HTTP client
- Symfony HTTP client
- framework HTTP clients
- shared outbound request wrappers

## 5.4 Redirect, protocol, DNS, and proxy source heuristics
Pay attention to:
- `CURLOPT_FOLLOWLOCATION`
- protocol restrictions
- stream wrapper support
- proxy/no-proxy handling
- DNS resolution and IP range checks
- helper defaults that expand allowed transports

## 5.5 Indirect and stored source heuristics
Pay attention to:
- webhook test paths
- previewers
- remote file/image loaders
- metadata/OpenGraph fetchers
- queue jobs using stored URLs
- cron/admin jobs that fetch remote targets
- saved integration or tenant endpoints

---

# 6. False-Positive Controls

Do not mark a PHP source as high-priority if:
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

Good PHP source evidence includes:
- route/controller/script/worker/admin/import/render entry point,
- source API such as `$_GET`, request input, uploaded metadata, queue payload, config record, or stored callback record,
- propagation such as URL construction, URL parsing, host/path recomposition, storage, redirect option setting, protocol option setting, proxy option construction, or wrapper call,
- cURL, `file_get_contents`, stream wrapper, Guzzle, framework HTTP client, renderer/importer, crawler, or network wrapper receiving the value,
- SSRF context when visible.

Good source evidence answers:
1. Which PHP entry point receives the outbound-request-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which direct fetch, URL construction, stored callback, redirect, DNS, protocol, proxy, wrapper, renderer, or worker behavior should be audited next?
4. Is the source used for full URL, host, IP, scheme, port, path, redirect target, proxy, metadata endpoint, internal-service target, stored endpoint, or indirect resource reference?

---

# 8. Quick PHP Source Checklist

- Are request values used as full URLs, callback targets, hosts, ports, paths, schemes, or remote resource references?
- Can request or stored values name localhost, loopback, link-local, private-network, cloud metadata, or internal service targets?
- Are stored webhook, callback, integration, or tenant endpoints later fetched?
- Are redirects, DNS names, parser output, or final resolved addresses influenced?
- Are proxy, protocol, stream wrapper, or client options dynamic?
- Are preview/import/render/crawler features fetching externally supplied resources?
- Are background jobs and retries revalidating stored targets before request use?
