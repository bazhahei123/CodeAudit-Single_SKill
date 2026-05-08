# SSRF Common Cases

## Purpose

This file defines shared server-side request forgery concepts, audit logic, anti-patterns, false-positive controls, and candidate sink search groups that apply across languages and frameworks.

Use this file as the base reference for SSRF review before loading any language-specific reference.

This reference is guidance, not proof. Do not report a vulnerability only because code resembles a pattern described here. Always verify the real data flow, real request target, final network destination, and missing protection in the target code.

---

# 1. Core Concepts

## 1.1 What SSRF risk is
SSRF risk exists when attacker-controlled or weakly trusted input can influence a server-side outbound request target, remote resource fetch, network connection, webhook dispatch, URL preview, import, parser, renderer, or SDK endpoint in a way that can reach unintended internal, privileged, or restricted destinations.

The core question is:

**Can attacker-controlled input influence where server-side code connects, resolves, redirects, fetches, or loads a network resource?**

## 1.2 Source, propagation, target construction, and sink

### Source
A source is any attacker-controllable input, including:
- query parameters
- path parameters
- request body fields
- headers
- cookies
- uploaded metadata
- URL fields
- webhook or callback targets
- import, preview, avatar, feed, or sitemap URLs
- stored URLs previously written by a user or admin
- queue payloads and replay jobs
- external service responses derived from user-controlled data

### Propagation
Propagation means the input is copied, decoded, normalized, concatenated, parsed, stored, resolved, redirected, proxied, or transformed before reaching an outbound request sink.

Do not assume a renamed, parsed, or initially validated URL is safe unless the final destination is constrained at request time.

### Request target construction
Request target construction is where code builds or chooses the scheme, host, port, path, query, proxy, endpoint, or full URL used by a network client.

### Sink
A sink is the place where the application performs outbound network behavior, including:
- HTTP/HTTPS client requests
- URL openers and stream wrappers
- socket connections
- FTP/SFTP or alternate protocols
- cloud SDK requests with dynamic endpoint overrides
- webhook dispatch
- browser/screenshot/rendering resource fetches
- XML/document/image/parser resource loading
- network client wrappers that hide the real request

---

# 2. Shared SSRF Attack Surfaces

Prioritize these attack surfaces first:
- public route/controller annotations and HTTP handler registration
- GraphQL resolvers and mutations that fetch, preview, verify, or import URLs
- RPC, gRPC, SOAP, WCF, Thrift, Binder, DBus, or internal protocol methods that carry URLs or hosts
- WebSocket and message-frame handlers carrying callback, webhook, or URL values
- Android exported components, WebView bridges, SDK callbacks, and content providers that trigger network fetches
- webhook and callback test endpoints
- URL preview, screenshot, crawler, or metadata fetch features
- remote file import or download helpers
- HTML, markdown, PDF, XML, SVG, image, office, or document renderers that load external resources
- image, avatar, feed, sitemap, OpenGraph, oEmbed, or link unfurl fetchers
- generic HTTP client wrapper services
- queue consumers, replay tools, retries, and scheduled jobs that fetch stored URLs
- admin tools that verify connectivity or fetch external resources
- cloud SDK calls with dynamic endpoint, region, bucket, or URL override values
- legacy helpers and alternate code paths to the same outbound client

## 2.1 Candidate search groups for graph workflows

When a graph database cannot start from a single universal sink, build candidate sets from four groups and intersect them.

### Entry candidates
Search for externally reachable or semi-trusted request entry points:
- controller
- route
- handler
- endpoint
- resolver
- mutation
- action
- servlet
- middleware
- webhook
- callback
- listener
- consumer
- worker
- job
- scheduler
- admin
- debug
- diagnostic
- connectivity test
- preview
- screenshot
- crawler
- fetch
- import
- download
- upload from URL
- avatar
- image
- feed
- sitemap
- oembed
- opengraph
- metadata
- render
- parser
- proxy
- bridge
- IPC
- RPC
- WebSocket
- deep link
- exported

### Request-target construction candidates
Search for untrusted values entering URL, host, endpoint, proxy, or DNS decisions:
- url
- uri
- endpoint
- target
- host
- hostname
- domain
- scheme
- protocol
- port
- path
- callbackUrl
- webhookUrl
- redirectUrl
- returnUrl
- imageUrl
- avatarUrl
- feedUrl
- importUrl
- downloadUrl
- previewUrl
- metadataUrl
- proxyUrl
- baseUrl
- endpointOverride
- regionEndpoint
- URL decode
- base64 decode
- URI parse
- URL parse
- DNS resolve
- InetAddress
- IPAddress
- redirect
- Location header

### Outbound request sink candidates
Search for APIs that open outbound network connections:
- request
- fetch
- get
- post
- put
- send
- open
- openConnection
- urlopen
- HttpClient
- RestTemplate
- WebClient
- OkHttp
- Guzzle
- cURL
- requests
- httpx
- aiohttp
- urllib
- socket
- TcpClient
- WebRequest
- HttpWebRequest
- URLSession
- libcurl
- Boost.Beast
- cloud SDK client
- endpoint override
- XML external entity fetch
- document renderer fetch
- screenshot browser navigation
- metadata fetcher
- webhook dispatcher

### Required-control candidates
Search near sinks for controls:
- allowlist
- whitelist
- permitted hosts
- fixed endpoint
- scheme allowlist
- host allowlist
- deny private IP
- block localhost
- block loopback
- block link-local
- block metadata
- block RFC1918
- resolved IP check
- DNS pinning
- re-resolve before connect
- redirect disabled
- redirect revalidation
- max redirects
- proxy disabled
- no_proxy
- timeout
- size limit
- content type allowlist
- final URL validation
- canonical host
- normalized host
- IDN handling
- trusted-only

## 2.2 Generic graph search recipes

Useful candidate recipes:

```text
<entry candidate> + <outbound request sink candidate>
<webhook/callback/preview/import/render> + <url/host/endpoint candidate>
<request/body/header/job payload> + <URL construction candidate>
<stored URL/webhook target> + <later request sink>
<redirect-following client> without <redirect revalidation>
<host allowlist> without <resolved IP check>
<cloud SDK endpoint override> + <user-controlled value>
<renderer/parser/browser> + <external resource loading>
<candidate request sink> without nearby <required-control candidate>
```

---

# 3. Shared Anti-Patterns

## A1. Direct fetch of attacker-controlled URL
High-risk pattern:
- request parameter, body field, or stored URL is passed directly to an HTTP client.

Why risky:
The attacker may target internal services, metadata endpoints, localhost, or private networks.

## A2. Scheme-only or substring validation
High-risk pattern:
- `startsWith("http")`
- `contains("trusted.com")`
- regex checks that do not parse and compare the final hostname.

Why risky:
Parser confusion, userinfo tricks, redirects, IDN, and suffix/prefix issues can bypass string checks.

## A3. Initial validation without final destination control
High-risk pattern:
- the first URL is validated, but redirects, DNS resolution, proxy behavior, or alternate clients change the final destination.

Why risky:
The final connection target matters more than the initial string.

## A4. Stored URL fetched later
High-risk pattern:
- webhook URL, callback target, feed URL, avatar URL, or import URL is stored and later fetched by a job.

Why risky:
Second-order SSRF paths are often missed if validation is only applied during creation or not rechecked before fetch.

## A5. Renderer or parser loads remote resources
High-risk pattern:
- HTML, markdown, SVG, XML, PDF, image, office, browser, or screenshot logic loads external URLs.

Why risky:
The visible code may not call a normal HTTP client, but the renderer still performs network requests.

## A6. Dynamic cloud SDK endpoint or proxy settings
High-risk pattern:
- endpoint override, proxy URL, region endpoint, bucket endpoint, or base URL is influenced by untrusted data.

Why risky:
SDKs can reach metadata, internal services, or attacker-selected endpoints.

---

# 4. False-Positive Controls

Do not report a vulnerability as `Confirmed` if:
- the target is provably constant or server-controlled,
- the apparent URL is not actually fetched,
- the final destination is restricted by a strict scheme and host allowlist,
- resolved IP checks block loopback, private, link-local, multicast, and metadata ranges at connect time,
- redirects are disabled or every redirect target is revalidated,
- the sink is unreachable from attacker-controlled input.

Use `Suspected` or `Not enough evidence` if:
- the sink exists but attacker influence on the target is unclear,
- input reaches URL construction but the final client wrapper is hidden,
- allowlist, DNS, redirect, proxy, or final-IP checks may exist elsewhere but cannot be verified,
- stored URLs may be dangerous but the later fetch path is not visible.

Do not over-claim based only on:
- an outbound HTTP client existing,
- variable names such as `url` or `callback`,
- a local helper named `safeFetch` without reading its implementation,
- initial parsing without evidence of final network destination control.
