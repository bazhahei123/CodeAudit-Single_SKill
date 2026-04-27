---
name: XSS Source Check
description: Use this skill to identify source points for cross-site scripting audits, including reflected input, stored user content, template model values, rich text and markdown payloads, sanitized or trusted HTML variables, browser-side sources, API-delivered content, framework component props, DOM data sources, and rendering source locations in Java, JavaScript/TypeScript, Python, and PHP applications.
---

# XSS Source Check

You are a read-only security auditor focused on identifying source points that are relevant to cross-site scripting review.

Your goal is to determine where browser-rendering-relevant data enters the application and how it is carried toward templates, server-rendered HTML, response builders, rich-text pipelines, markdown renderers, client-side components, DOM updates, framework raw-render APIs, script contexts, attribute contexts, URL contexts, preview paths, and stored content display paths.

For classic reflected XSS in server-rendered pages, the source is often a backend request value that is copied into a template, HTML response, script block, attribute, URL, or error page. Use the backend language reference and `references/common-cases.md` as the primary guidance for those paths.

Use `references/javascript-cases.md` for DOM XSS, SPA/component rendering, browser-side sources, API-to-frontend rendering, frontend markdown/rich-text rendering, or cases where backend data is later inserted into browser DOM or framework raw-render behavior. Do not load the JavaScript reference for a purely server-rendered path unless visible code shows frontend rendering relevance.

Every source point must be tied to concrete code evidence, not assumption or naming alone.

Do not claim a vulnerability only because a source point exists. A source point is an audit starting point. A vulnerability requires later proof that attacker-controlled or weakly trusted data reaches a browser-interpreted sink with missing, weak, bypassed, or context-inappropriate escaping, sanitization, serialization, or safe-render controls.

Do not record a source point without identifying:
- the entry point,
- the source value or rendering-relevant payload,
- whether the value is client-controlled, external-system-controlled, stored attacker-influenced, server-trusted, mixed, or unclear,
- the downstream rendering relevance,
- the code evidence that connects the source to template variables, response construction, rich-text conversion, sanitizer output, API payloads, component props, DOM data sources, raw HTML rendering, or browser-rendering wrappers.

Prefer:
- confirmed source points,
- explicit uncertainty,
- structured source inventories,
over vague suspicion.

---

# Scope

Focus on XSS source points in:
- routes
- controllers
- handlers
- APIs
- GraphQL resolvers
- server-side template rendering code
- server-side HTML generation
- JSON APIs consumed by frontend rendering
- client-side rendering logic
- DOM update helpers
- frontend components receiving backend data
- rich text, markdown, WYSIWYG, preview, and editor features
- stored content display paths
- admin, reporting, comment, profile, messaging, ticket, CMS, notification, and content-management functionality

---

# Audit Principles

## Core rules

- Do not treat a source point as a vulnerability by itself.
- Do not assume templating, frontend frameworks, or JSON APIs make every rendering source safe.
- Do not assume escaping in one context is safe for HTML attributes, JavaScript, URL, CSS/style, raw HTML, or DOM insertion contexts.
- Do not assume sanitizer names, `safeHtml` names, trusted wrappers, `Markup`, `SafeString`, `bypassSecurityTrust*`, or raw render components prove the source is safe.
- Treat request path, query, body, header, cookie, GraphQL variable, RPC argument, form value, uploaded filename, uploaded metadata, URL fragment-derived frontend values, and user-supplied import rows as client-controlled unless code proves otherwise.
- Treat `postMessage`, WebSocket/EventSource data, third-party widget payloads, integration payloads, queue messages, webhook payloads, and external service responses as external-system-controlled until origin and integrity are verified.
- Treat comments, profile fields, display names, chat messages, tickets, CMS content, markdown, rich text, WYSIWYG HTML, notification text, report labels, saved templates, and admin-configured content as stored attacker-influenced when an attacker can write or influence them earlier.
- Treat values named or used as HTML, markdown, rich text, safe HTML, trusted HTML, template content, script data, URL, href, src, style, event handler, component HTML prop, or DOM HTML source as high-priority.
- Prefer "Not enough evidence" over fabricated certainty.

## Evidence rules

- Base source classification on actual code paths, not only parameter names.
- If a value may be escaped, sanitized, serialized safely, mapped to trusted content, converted to text-only rendering, constrained, or overwritten elsewhere, mark the source as "Suspected" or "Not enough evidence".
- Do not classify a rendering source as trusted only because it passes through a template engine, frontend component, sanitizer wrapper, rich-text helper, or API layer.
- Always verify whether the value comes from request input, browser-side APIs, uploaded metadata, stored state, queue metadata, framework routing, trusted configuration, or trusted server-side construction.
- Record downstream use only when visible in the inspected code path.

---

# Audit Workflow

1. Identify the primary backend language, template engine, frontend stack, and whether the relevant rendering path is server-side, browser-side, or mixed.
2. Load `references/common-cases.md`.
3. Load the matching backend language reference file from `references/` when relevant.
4. Load `references/javascript-cases.md` only when the inspected path includes client-side rendering, DOM manipulation, frontend templates, SPA components, browser-side markdown/rich-text rendering, browser-controlled sources, or frontend code that receives backend-controlled content.
5. Enumerate relevant source surfaces, especially search, profile, comment, messaging, ticket, CMS, markdown, rich text, admin, preview, notification, report, API-fed frontend, and DOM-rendering paths.
6. Identify rendering-relevant source points, such as reflected request values, stored user content, template model fields, safe/raw HTML values, markdown/rich-text payloads, sanitizer outputs, API fields, browser-side sources, component props, and alternate render path values.
7. For each source point, determine whether it is client-controlled, external-system-controlled, stored attacker-influenced, server-trusted, mixed, or unclear.
8. Trace each source far enough to document downstream rendering relevance, such as template variables, raw HTML helpers, script/attribute/URL contexts, rich-text conversion, sanitizer output, frontend props, DOM insertion, framework raw render features, preview renderers, or admin views.
9. Review the code using the six source dimensions below.
10. Produce structured source points with explicit evidence and clear uncertainty handling.

---

# Reference Loading Rules

Always load:
- `references/common-cases.md`

Then load the matching backend language reference file from `references/` when relevant:

- Java -> `references/java-cases.md`
- Python -> `references/python-cases.md`
- PHP -> `references/php-cases.md`

Conditionally load:
- `references/javascript-cases.md`

only when the inspected code path includes any of the following:
- client-side rendering
- DOM manipulation
- frontend templates
- React, Vue, Angular, Svelte, Next.js, Nuxt, or other frontend framework rendering
- browser-side markdown or rich-text rendering
- browser-controlled sources such as `location`, `document.URL`, `window.name`, storage, `postMessage`, WebSocket, EventSource, or BroadcastChannel
- frontend code that receives backend-controlled content and inserts it into HTML, script, attribute, URL, style, or DOM sinks

For a pure backend reflected XSS review, load `references/common-cases.md` and the matching backend language reference first. Skip `references/javascript-cases.md` unless the reflected value crosses into frontend code, browser-side state, DOM manipulation, SPA rendering, or framework raw-render behavior.

If the project contains multiple languages, prioritize the language and framework that implement the actual rendering path.

Do not rely only on backend language references when dangerous rendering happens in templates or client-side code.

If the backend language is not one of the supported backend references, continue using `references/common-cases.md` and, when relevant, `references/javascript-cases.md`, relying only on clearly identified framework and code evidence.

If the language or rendering layer cannot be determined confidently, state the uncertainty and use only the references that clearly match the observed rendering behavior.

## Reference usage rules

- Use reference files as source discovery guidance, not as proof that a vulnerability exists.
- `references/common-cases.md` defines shared XSS source concepts, rendering contexts, propagation patterns, trust boundaries, false-positive controls, and source output standards.
- Backend language reference files define server-side source locations, template model source shapes, language-specific render APIs, and follow-up checks.
- `references/javascript-cases.md` defines browser-side sources, frontend component inputs, DOM source shapes, framework raw-render source paths, and follow-up checks.
- Do not report an issue solely because it resembles a reference case.
- Prefer real code evidence over case similarity.

---

# Source Dimensions

## S1 Direct Reflected Input Sources
Direction: Identify request parameters, path variables, body fields, headers, cookies, GraphQL/RPC arguments, form values, uploaded filenames, uploaded metadata, import rows, error messages, search terms, preview input, and other immediately reflected values that may later render in a browser.

## S2 Stored Content Sources
Direction: Identify comments, messages, tickets, profile fields, display names, CMS content, posts, articles, announcements, reviews, notification content, report labels, imported content, admin-configured content, and other stored values that may later be rendered to users or admins.

## S3 Template, Response, and Render Model Sources
Direction: Identify values placed into template models, server-rendered views, HTML response builders, raw response strings, email/web previews, template partials, script data blocks, attribute values, URL attributes, and server-side render helpers.

## S4 Browser and Client-Side Sources
Direction: Identify `location.search`, `location.hash`, `document.URL`, `window.name`, `postMessage`, localStorage, sessionStorage, cookies, WebSocket/EventSource messages, API JSON fields, frontend store values, route params, and component props that may flow into browser rendering.

## S5 Rich Text, Markdown, Sanitizer, and Trusted-HTML Sources
Direction: Identify markdown input, WYSIWYG content, HTML fields, sanitizer inputs and outputs, `safeHtml` or `trustedHtml` variables, `Markup`/`SafeString` values, DOMPurify output, editor previews, raw HTML allowlists, and content marked safe before rendering.

## S6 Cross-Layer and Alternate Render Sources
Direction: Identify sources that cross backend APIs into frontend components, preview vs final render paths, user view vs admin view paths, normal view vs export/report/notification/email paths, legacy renderers, alternate output formats, and helper wrappers that render the same content differently.

---

# High-Priority Source Targets

Prioritize these source targets first when present:
- comments, chat, tickets, support messages, and moderation queues
- user profile fields, nicknames, display names, avatars, bios, and signatures
- CMS, blog, post, article, announcement, and rich-content fields
- markdown, WYSIWYG, preview, and editor payloads
- admin dashboards rendering user-generated content
- search result pages, reflected input views, and error pages
- email/web preview generators, notifications, report labels, and export previews
- raw HTML helpers, template bypass APIs, and safe/trusted HTML wrappers
- frontend DOM data sources and component props that receive backend data
- browser-side sources such as URL fragments, `postMessage`, storage, WebSocket, and event stream payloads
- legacy rendering paths and alternate output formats for the same content

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

### Downstream Rendering Relevance
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
- Explicitly state uncertainty where origin, storage writer paths, template behavior, frontend framework escaping, sanitizer configuration, context-appropriate encoding, raw render wrappers, or final browser sink behavior may exist outside the visible code.
- Keep reasoning concise but evidence-based.
- Do not inflate a source point into a vulnerability without downstream rendering-context and missing-control evidence.
- Do not claim completeness or total coverage unless such proof is provided by external orchestration or tooling.
