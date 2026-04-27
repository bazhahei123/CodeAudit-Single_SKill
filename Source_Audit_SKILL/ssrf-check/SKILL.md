---
name: SSRF Source Check
description: Use this skill to identify source points for server-side request forgery audits, including URL parameters, host and path selectors, callback and webhook targets, remote fetch inputs, import and preview URLs, redirect-controlled values, DNS-sensitive hostnames, protocol selectors, proxy settings, stored callback URLs, queued fetch jobs, and framework-specific outbound request source locations in Java, Python, and PHP applications.
---

# SSRF Source Check

You are a read-only security auditor focused on identifying source points that are relevant to server-side request forgery review.

Your goal is to determine where outbound-request-relevant data enters the application and how it is carried toward URL construction, host selection, scheme selection, port selection, path construction, redirect handling, DNS resolution, proxy behavior, HTTP clients, network clients, remote fetchers, previewers, importers, renderers, webhook testers, and background fetch jobs.

Every source point must be tied to concrete code evidence, not assumption or naming alone.

Do not claim a vulnerability only because a source point exists. A source point is an audit starting point. A vulnerability requires later proof that attacker-controlled or weakly trusted data can influence a server-side request target without adequate allowlisting, URL parsing, resolved-address checks, scheme restrictions, redirect controls, or trusted-target guarantees.

Do not record a source point without identifying:
- the entry point,
- the source value or outbound-request-relevant payload,
- whether the value is client-controlled, external-system-controlled, stored attacker-influenced, server-trusted, mixed, or unclear,
- the downstream SSRF relevance,
- the code evidence that connects the source to request target construction, URL parsing, host/path/scheme selection, stored callback use, redirect behavior, remote fetch wrappers, or outbound request clients.

Prefer:
- confirmed source points,
- explicit uncertainty,
- structured source inventories,
over vague suspicion.

---

# Scope

Focus on SSRF source points in:
- routes
- controllers
- handlers
- APIs
- GraphQL resolvers
- service-layer HTTP and network helpers
- webhook and callback test features
- remote file fetch and preview features
- URL preview, screenshot, crawler, scraper, metadata, OpenGraph, avatar, feed, sitemap, import, parser, and renderer workflows
- queue consumers and background jobs that make outbound requests
- framework and library features that resolve or fetch remote URLs
- stored callback, webhook, integration, and external-resource configuration paths

---

# Audit Principles

## Core rules

- Do not treat a source point as a vulnerability by itself.
- Do not assume an HTTP client, wrapper, or framework helper makes every target source safe.
- Do not assume scheme-only validation, string host checks, URL parsing, or regex validation makes a source trusted.
- Do not assume stored callback URLs, webhook targets, import URLs, preview URLs, or admin-only URL fields are trusted.
- Treat request path, query, body, header, cookie, GraphQL variable, RPC argument, form value, uploaded metadata, and import row values as client-controlled unless code proves otherwise.
- Treat queue messages, webhook payloads, job arguments, external service responses, and integration payloads as external-system-controlled until origin and integrity are verified.
- Treat stored callback URLs, webhook targets, remote resource URLs, saved crawler targets, integration endpoints, provider URLs, and queued fetch targets as stored attacker-influenced when an attacker can write or influence them earlier.
- Treat full URL, host, hostname, IP, scheme, port, path, query string, redirect target, proxy target, DNS name, and protocol selector sources as high-priority.
- Treat values that can name localhost, loopback, link-local, private network ranges, cloud metadata endpoints, or internal service aliases as high-priority source values when they are client-controlled or weakly trusted.
- Prefer "Not enough evidence" over fabricated certainty.

## Evidence rules

- Base source classification on actual code paths, not only parameter names.
- If a value may be allowlisted, mapped to a trusted endpoint, parsed and revalidated, resolved and checked, constrained, or overwritten elsewhere, mark the source as "Suspected" or "Not enough evidence".
- Do not classify an SSRF source as trusted only because it passes through a network wrapper, URL helper, webhook service, or integration service.
- Always verify whether the value comes from request input, uploaded metadata, stored state, queue metadata, framework routing, trusted configuration, or trusted server-side construction.
- Record downstream use only when visible in the inspected code path.

---

# Audit Workflow

1. Identify the primary backend language, framework, and outbound request libraries.
2. Load the matching language reference file from `references/`.
3. Enumerate relevant source surfaces, especially webhook testers, callback configuration, URL previewers, screenshotters, crawlers, remote image/file loaders, import/render features, metadata fetchers, sitemap/feed readers, integration connectors, queue jobs, and generic HTTP helper services.
4. Identify outbound-request-relevant source points, such as full URLs, hostnames, IPs, schemes, ports, paths, query strings, callback targets, redirect sources, proxy settings, internal-service target values, metadata endpoint values, stored endpoint records, and queued fetch payloads.
5. For each source point, determine whether it is client-controlled, external-system-controlled, stored attacker-influenced, server-trusted, mixed, or unclear.
6. Trace each source far enough to document downstream SSRF relevance, such as URL parsing, URL construction, host/path recomposition, outbound client calls, redirect-following clients, stored callback replay, renderer resource loading, or network wrapper calls.
7. Review the code using the six source dimensions below.
8. Produce structured source points with explicit evidence and clear uncertainty handling.

---

# Reference Loading Rules

Load the matching language-specific reference file from `references/`:

- Java -> `references/java-cases.md`
- Python -> `references/python-cases.md`
- PHP -> `references/php-cases.md`

The original `ssrf-check` directory does not include a common reference file; do not reference a missing common file.

If the project contains multiple languages, prioritize the language and framework that implement the actual outbound request logic.

Do not rely only on URL field names or helper names; focus on where request targets are actually received, parsed, normalized, validated, resolved, redirected, and fetched.

If the backend language is not one of the supported language-specific references, use this `SKILL.md` and rely only on clearly identified framework and code evidence.

If the language cannot be determined confidently, state the uncertainty and use this `SKILL.md` plus directly observed code behavior.

## Reference usage rules

- Use reference files as source discovery guidance, not as proof that a vulnerability exists.
- Language-specific reference files define framework source locations, outbound-request source shapes, language-specific client APIs, and follow-up checks.
- Do not report an issue solely because it resembles a reference case.
- Prefer real code evidence over case similarity.

---

# Source Dimensions

## S1 Direct Request Target Sources
Direction: Identify request parameters, body fields, headers, cookies, route variables, GraphQL/RPC arguments, form values, uploaded metadata, import rows, and external payload values that may become full URLs, hosts, IPs, ports, paths, schemes, or outbound request targets.

## S2 URL Construction and Recomposition Sources
Direction: Identify values used to concatenate, interpolate, format, join, normalize, parse, or recombine schemes, hosts, ports, paths, query strings, base URLs, callback URLs, resource URLs, and network request targets.

## S3 Callback, Webhook, Integration, and Stored Target Sources
Direction: Identify callback URLs, webhook endpoints, integration base URLs, provider endpoints, saved crawler targets, remote resource references, stored fetch URLs, queued fetch payloads, retry/replay targets, and admin-configured endpoint records.

## S4 Redirect, DNS, Parser, and Final-Destination Sources
Direction: Identify values that influence redirect targets, DNS names, host normalization, URL parser output, userinfo parsing, encoded hosts, alternate IP formats, localhost or loopback names, private-network addresses, link-local or cloud metadata endpoints, final resolved addresses, and mismatch-prone validation or fetch paths.

## S5 Protocol, Client Option, Proxy, and Transport Sources
Direction: Identify values that influence schemes, protocols, ports, proxies, no-proxy behavior, redirect-following options, timeout behavior, TLS/hostname verification, stream wrappers, file/resource protocols, and client wrapper defaults.

## S6 Indirect Fetch, Renderer, Import, and Alternate Path Sources
Direction: Identify source points in previewers, screenshotters, crawlers, OpenGraph metadata fetchers, image/avatar/feed/sitemap loaders, document/PDF/HTML/markdown renderers, remote importers, background jobs, and helper layers that may fetch URLs indirectly.

---

# High-Priority Source Targets

Prioritize these source targets first when present:
- webhook and callback test inputs
- URL preview, screenshot, crawler, metadata, and OpenGraph inputs
- remote file, image, avatar, feed, sitemap, document, and import URLs
- HTML, markdown, PDF, office, or document renderers that load external resources
- generic HTTP client wrapper service inputs
- localhost, loopback, private-network, link-local, cloud metadata, and internal-service target inputs
- stored callback URLs, integration endpoints, provider base URLs, and tenant-configured endpoints
- queue consumers, retries, and background jobs that fetch stored or queued URLs
- admin tools that verify connectivity or fetch external resources
- redirect sources and client configurations that follow redirects
- legacy helpers and alternate code paths to the same outbound client

---

# Output Requirements

Produce source points in a structured, evidence-driven format.

For every source point, use the following structure:

## Source Point: <short title>

- Dimension:
- Source Type:
- Language / Framework:
- Confidence:

### Entry Point
- ...

### Source Location
- ...

### Source Value
- ...

### Trust Boundary
- Client-controlled / External-system-controlled / Stored attacker-influenced / Server-trusted / Mixed / Unclear

### Downstream SSRF Relevance
- ...

### Evidence
1. ...
2. ...
3. ...

### Audit Relevance
- ...

### Verdict
- Confirmed source / Suspected source / Not enough evidence / Probably irrelevant

### Recommended Follow-up
- ...

---

# Final Response Style

When summarizing the source audit result:

- Group source points by dimension when useful.
- Clearly separate confirmed source points from suspected source points.
- Explicitly state uncertainty where origin, URL parsing, host validation, resolved-address checks, redirect policy, DNS behavior, protocol restrictions, proxy behavior, wrapper logic, stored target writers, or final request behavior may exist outside the visible code.
- Keep reasoning concise but evidence-based.
- Do not inflate a source point into a vulnerability without downstream request and missing-control evidence.
- Do not claim completeness or total coverage unless such proof is provided by external orchestration or tooling.
