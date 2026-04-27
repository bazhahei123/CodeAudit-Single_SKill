# Common SSRF Source Cases

## Purpose

This file defines shared source point guidance for SSRF source discovery across languages and frameworks.

Use it before language-specific references to identify:
- where outbound-request-relevant values enter the application,
- who controls those values,
- how they are transformed before request use,
- which downstream SSRF controls should be audited next.

This reference is guidance, not proof. A source point is an audit starting point, not a vulnerability by itself.

---

# 1. Core SSRF Source Concepts

## 1.1 Outbound-request-relevant source point

An SSRF source point is where data enters or becomes available to the application and may influence a server-side network request target.

Common source values include:
- full URL
- hostname
- IP address
- scheme or protocol
- port
- path
- query string
- callback URL
- webhook URL
- import URL
- preview URL
- avatar or image URL
- feed or sitemap URL
- proxy target
- redirect target
- internal-service target
- tenant or integration endpoint
- queued fetch target
- stored callback or webhook record

Do not require a direct HTTP client call at the source point. It is enough that the value can later reach URL construction, parsing, resolution, redirect handling, a network wrapper, a renderer, an importer, or a background fetch job.

## 1.2 Source origin

Record where the value originates:
- request path, query, body, header, cookie, GraphQL variable, RPC argument, or form field
- uploaded metadata or imported row
- external webhook, callback, queue message, provider event, or integration payload
- stored callback URL, webhook target, tenant endpoint, provider endpoint, crawler target, or remote resource reference
- admin-configured endpoint or test URL
- framework route value, config value, or service discovery record
- renderer, parser, crawler, or importer option

## 1.3 Propagation

Trace how the source is carried:
- copied into URL strings
- split into host, path, scheme, port, or query components
- normalized or parsed by a URL helper
- recomposed after validation
- stored and fetched later by a worker
- passed through a generic HTTP client wrapper
- passed into a renderer, crawler, scraper, previewer, importer, or metadata fetcher
- used to configure redirect, proxy, resolver, TLS, or transport behavior

## 1.4 Downstream SSRF relevance

A source point is SSRF-relevant when it can influence:
- final network destination
- host or IP resolution
- scheme or protocol
- port or transport behavior
- path or query sent to an internal service
- redirect-following behavior
- proxy or no-proxy routing
- renderer or parser resource loading
- stored target replay by jobs, retries, callbacks, or integrations

---

# 2. Trust Boundary Classification

## Client-controlled

Use this when the value comes from:
- request parameters
- request body fields
- path variables
- headers
- cookies
- GraphQL variables
- RPC arguments
- uploaded metadata
- import rows supplied by users
- form fields for preview, callback, webhook, import, or connectivity tests

## External-system-controlled

Use this when the value comes from:
- webhook payloads
- callback events
- queue messages
- provider events
- partner payloads
- third-party sync data
- integration responses
- external metadata documents

External-system-controlled values are not automatically trusted. Record what proves the external system is authenticated, integrity-protected, and authorized to choose the outbound target.

## Stored attacker-influenced

Use this when the value is loaded from:
- database records
- tenant settings
- integration endpoint records
- webhook or callback registrations
- saved crawler targets
- import job records
- report or preview records
- cache entries
- queue retry payloads
- object storage metadata

Stored values are source points when an attacker may have written or influenced them earlier.

## Server-trusted

Use this only when code evidence shows the value comes from:
- fixed server-side constants
- strict server-side endpoint mappings
- trusted configuration not influenced by tenants or users
- service discovery records controlled by the deployment
- verified internal routing rules
- recomputed values that ignore untrusted input

## Mixed

Use this when a trusted base is combined with an untrusted component, such as:
- fixed base URL plus user-controlled path
- trusted provider plus user-controlled callback path
- tenant endpoint plus request-controlled resource path
- allowlisted host plus user-controlled scheme or port
- validated initial URL plus unvalidated redirect target

## Unclear

Use this when source origin or writer path cannot be proven from visible code.

---

# 3. Shared SSRF Source Categories

## 3.1 Direct URL sources

Look for values named or used as:
- `url`
- `uri`
- `target`
- `endpoint`
- `callback`
- `callbackUrl`
- `webhookUrl`
- `redirectUrl`
- `imageUrl`
- `avatarUrl`
- `importUrl`
- `feedUrl`
- `sitemapUrl`
- `previewUrl`
- `remoteUrl`

Source relevance:
The value may directly determine the server-side request target.

Follow-up:
Trace whether the final request uses strict target allowlists, scheme restrictions, redirect controls, and resolved-address checks.

## 3.2 Host, IP, scheme, port, path, and query sources

Look for partial URL components:
- host
- hostname
- IP
- domain
- scheme
- protocol
- port
- path
- route
- query
- base URL
- provider endpoint
- service name

Source relevance:
Partial target control can still affect the final destination after URL recomposition.

Follow-up:
Verify that validation applies to the final canonical request target, not only to individual pieces before recomposition.

## 3.3 Callback, webhook, and integration target sources

Look for:
- webhook registration endpoints
- callback URL settings
- integration base URL settings
- notification endpoint settings
- tenant-configured connector targets
- provider callback replay logic
- webhook test features

Source relevance:
Stored or externally supplied endpoints can be fetched later by trusted server-side jobs.

Follow-up:
Identify writer path, storage path, test path, retry path, and final delivery path. Check whether each use revalidates the final target.

## 3.4 Preview, import, crawler, and renderer sources

Look for:
- URL preview
- screenshot generation
- OpenGraph or metadata fetch
- remote image loading
- avatar fetching
- feed or sitemap import
- document, HTML, Markdown, PDF, or office rendering
- scraper or crawler jobs
- remote template or asset loading

Source relevance:
Indirect fetch features often hide the final network call behind a library or worker.

Follow-up:
Trace library behavior, external resource loading, redirect handling, protocol support, and network restrictions.

## 3.5 Redirect, DNS, and parser-sensitive sources

Look for values that influence:
- initial URL before redirect
- redirect target
- DNS name
- host normalization
- userinfo section
- encoded host
- alternate IP format
- parser-specific URL object
- final resolved address

Source relevance:
The value may pass initial validation but resolve or redirect to a restricted destination later.

Follow-up:
Inspect whether the application validates the final destination after parsing, redirects, DNS resolution, and canonicalization.

## 3.6 Protocol, proxy, and transport option sources

Look for values that influence:
- scheme
- protocol
- proxy
- no-proxy rules
- redirect-following mode
- timeout behavior
- TLS or hostname verification
- resolver behavior
- client wrapper options
- stream wrapper behavior

Source relevance:
Client options can expand which protocols or networks are reachable.

Follow-up:
Verify allowed schemes, proxy constraints, redirect policy, TLS behavior, and wrapper defaults.

## 3.7 Internal-service and metadata target sources

Look for values that can name or map to:
- localhost
- loopback addresses
- private network ranges
- link-local addresses
- cloud metadata endpoints
- Kubernetes or container service names
- internal admin services
- service registry aliases
- tenant-specific internal endpoints

Source relevance:
The source can become high-risk if it influences a request that reaches internal or privileged services.

Follow-up:
Check whether internal targets are blocked after real resolution and after redirects.

---

# 4. Shared Source Patterns

## S1. Request value becomes full URL

Pattern:
- a request field is passed as a complete URL to a fetcher, previewer, importer, webhook tester, or HTTP wrapper.

Record as source when:
- the value is client-controlled or external-system-controlled,
- it reaches URL parsing, request construction, or a wrapper call.

## S2. Request value controls URL component

Pattern:
- request fields separately control host, path, scheme, port, query, or resource ID used to build the final URL.

Record as source when:
- the component influences the final target,
- validation is not visibly tied to the final canonical request target.

## S3. Stored endpoint is fetched later

Pattern:
- a callback, webhook, integration endpoint, or tenant target is stored and later used by a worker or retry path.

Record as source when:
- the writer path may be attacker-influenced,
- the fetch path uses the stored value or derived URL.

## S4. External event supplies target

Pattern:
- a webhook, queue message, provider event, or partner payload contains a URL or endpoint that the server fetches.

Record as source when:
- event authenticity, authorization, or target constraints are not visible,
- the event value reaches a request target.

## S5. Renderer or parser fetches external resources

Pattern:
- a renderer, parser, document converter, screenshotter, crawler, or metadata extractor loads remote resources from user-supplied content or URLs.

Record as source when:
- external resource loading is possible,
- the value can influence what the server-side renderer fetches.

## S6. Redirect or DNS behavior changes final target

Pattern:
- code validates one URL form but the client follows redirects, resolves DNS later, or uses a different parser for the final request.

Record as source when:
- user-controlled values influence redirectable or DNS-sensitive targets,
- final destination validation is not visible.

## S7. Proxy or protocol input changes network path

Pattern:
- source controls proxy, scheme, protocol, no-proxy, or client option values.

Record as source when:
- the option changes reachable destination, protocol support, or routing behavior.

---

# 5. False-Positive Controls

Do not mark a source as high-priority if:
- the final target is selected only from strict server-side constants,
- the source is mapped to a fixed endpoint by a closed allowlist,
- the value affects only display text, logging, analytics labels, or non-network metadata,
- the value never reaches URL construction, URL parsing, request wrappers, network clients, renderers, importers, crawlers, or background fetch logic,
- stored endpoints are trusted-only and cannot be influenced by users, tenants, external systems, or imported data.

Use `Suspected source` or `Not enough evidence` if:
- writer paths for stored targets are missing,
- wrapper behavior is hidden,
- redirect handling is unclear,
- DNS or resolved-address checks are not visible,
- validation may exist in another layer,
- framework or library resource loading behavior is not visible,
- only naming suggests URL behavior and there is no observed propagation.

Do not classify a source as safe only because:
- it passes through a URL parser,
- it has an `http` or `https` scheme check,
- it is stored in the database,
- it is configured by an admin,
- it is used by a shared HTTP wrapper,
- it is called a webhook, callback, integration, preview, or import helper.

---

# 6. Source Classification

## Confirmed source

Use `Confirmed source` when there is clear evidence that:
- an outbound-request-relevant value enters the application,
- the source is client-controlled, external-system-controlled, stored attacker-influenced, mixed, or unclear,
- the value reaches URL construction, parsing, target selection, network wrappers, renderers, importers, redirect behavior, proxy behavior, or outbound request clients.

## Suspected source

Use `Suspected source` when:
- the source appears to influence an outbound request,
- but some propagation, writer path, wrapper behavior, or final request behavior is not visible.

## Not enough evidence

Use `Not enough evidence` when:
- source origin is unclear,
- the source name suggests URL behavior but propagation is not visible,
- downstream request use may exist but cannot be proven,
- another layer may overwrite or constrain the value.

## Probably irrelevant

Use `Probably irrelevant` when the value:
- is trusted server-side constant data,
- is selected by a closed allowlist into fixed targets,
- affects only non-network behavior,
- never reaches any network, renderer, importer, crawler, URL parser, request wrapper, or outbound client path.

---

# 7. What Good Evidence Looks Like

Strong SSRF source point records include:
- entry point such as route, handler, API, GraphQL resolver, queue consumer, admin tool, webhook receiver, importer, renderer, or job
- source value such as full URL, host, scheme, port, path, callback, webhook target, proxy, redirect target, or stored endpoint
- trust boundary such as client-controlled, external-system-controlled, stored attacker-influenced, server-trusted, mixed, or unclear
- propagation through URL parsing, URL construction, storage, recomposition, wrapper calls, renderer options, redirect settings, proxy settings, or job payloads
- downstream SSRF relevance such as request target selection, final destination, redirect behavior, DNS resolution, protocol handling, proxy routing, or indirect resource loading
- uncertainty where allowlists, final resolved-address checks, redirect policy, or wrapper controls may exist outside visible code

Good source evidence answers:
1. Where does the outbound-request-relevant value first enter?
2. Who can control or influence it?
3. What URL, host, scheme, port, path, callback, webhook, proxy, redirect, or internal target can it affect?
4. How far can it be traced toward request construction or network behavior?
5. Which sink-side controls should be audited next?

---

# 8. Quick SSRF Source Checklist

- Are request values used as full URLs or URL components?
- Are callback, webhook, integration, or tenant endpoints submitted, stored, tested, retried, or replayed?
- Are preview, screenshot, crawler, metadata, import, render, image, avatar, feed, sitemap, or document features fetching remote resources?
- Can any source name localhost, loopback, link-local, private-network, cloud metadata, or internal service targets?
- Are redirects followed, and can the source affect the redirected final target?
- Are DNS names, parser output, encoded hosts, userinfo, or alternate IP formats part of the source path?
- Can the source influence scheme, protocol, port, proxy, no-proxy, TLS, or client wrapper options?
- Are queue jobs or background workers fetching stored or external URLs?
- Is the source only mapped to fixed server-side endpoints, or can it alter the final request target?
- Is uncertainty clearly recorded instead of assuming either safety or vulnerability?
