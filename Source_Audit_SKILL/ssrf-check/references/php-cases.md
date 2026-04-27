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

# 2. PHP Source Patterns

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

# 3. Case Templates

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

# 4. PHP-Specific Audit Heuristics

## 4.1 Request and framework source heuristics
Pay attention to:
- `$_GET`, `$_POST`, and `$_REQUEST`
- `$request->input(...)`
- route parameters
- validation result arrays later used in HTTP helpers
- uploaded metadata and import rows
- webhook, preview, import, render, and admin route parameters

## 4.2 URL assembly source heuristics
Pay attention to:
- string-built URLs
- `parse_url(...)` parsing and recomposition
- host/path/base URL composition
- callback URL generation
- preview/import target construction
- helper methods named `fetch`, `download`, `preview`, `verify`, `crawl`, or `import`

## 4.3 Client API source heuristics
Pay attention to:
- cURL
- `file_get_contents` with URLs
- stream wrappers
- Guzzle
- Laravel HTTP client
- Symfony HTTP client
- framework HTTP clients
- shared outbound request wrappers

## 4.4 Redirect, protocol, DNS, and proxy source heuristics
Pay attention to:
- `CURLOPT_FOLLOWLOCATION`
- protocol restrictions
- stream wrapper support
- proxy/no-proxy handling
- DNS resolution and IP range checks
- helper defaults that expand allowed transports

## 4.5 Indirect and stored source heuristics
Pay attention to:
- webhook test paths
- previewers
- remote file/image loaders
- metadata/OpenGraph fetchers
- queue jobs using stored URLs
- cron/admin jobs that fetch remote targets
- saved integration or tenant endpoints

---

# 5. False-Positive Controls

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

# 6. What Good Evidence Looks Like

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

# 7. Quick PHP Source Checklist

- Are request values used as full URLs, callback targets, hosts, ports, paths, schemes, or remote resource references?
- Can request or stored values name localhost, loopback, link-local, private-network, cloud metadata, or internal service targets?
- Are stored webhook, callback, integration, or tenant endpoints later fetched?
- Are redirects, DNS names, parser output, or final resolved addresses influenced?
- Are proxy, protocol, stream wrapper, or client options dynamic?
- Are preview/import/render/crawler features fetching externally supplied resources?
- Are background jobs and retries revalidating stored targets before request use?
