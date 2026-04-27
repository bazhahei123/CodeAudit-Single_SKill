# XSS Common Cases

## Purpose

This file defines shared cross-site scripting concepts, audit logic, anti-patterns, false-positive controls, and finding standards that apply across languages, templates, and rendering frameworks.

Use this file as the base reference for XSS review before loading any language-specific reference.

This file explains:
- what XSS is,
- what counts as a source, sink, context, and control,
- how to distinguish reflected, stored, and DOM XSS,
- why output context matters,
- when to report `Confirmed`, `Suspected`, or `Not enough evidence`.

This reference is guidance, not proof. Do not report a vulnerability only because code resembles a pattern described here. Always verify the real data flow and real rendering sink in the target code.

---

# 1. Core Concepts

## 1.1 What XSS is
Cross-site scripting occurs when untrusted input is rendered into a browser-executable context in an unsafe way.

The core question is:

**Can attacker-controlled input reach a rendering or execution sink where the browser interprets it as HTML, script, markup, or active content rather than inert data?**

## 1.2 Source, propagation, and sink

### Source
A source is any attacker-controllable input, including:
- query parameters
- path parameters
- request body fields
- headers
- cookies
- uploaded filenames or metadata
- profile fields
- comments
- chat messages
- markdown or rich text input
- stored content previously written by a user or admin
- API responses derived from user-controlled content

### Propagation
Propagation means the input is copied, normalized, transformed, concatenated, wrapped in objects, or stored and later reloaded before reaching a sink.

Do not assume a renamed variable is safe.

### Sink
A sink is a place where the browser may interpret the data as active content, including:
- server-side template output
- raw HTML rendering
- DOM HTML insertion
- inline JavaScript contexts
- event handler attributes
- URL-based execution contexts
- style or CSS injection contexts that may become script-relevant
- preview or rich-content renderers

The main audit goal is to determine whether attacker-controlled input reaches an unsafe browser-interpreted sink.

## 1.3 Reflected, stored, and DOM XSS

### Reflected XSS
Attacker input is returned immediately in the response and rendered unsafely.

Examples:
- search query echoed into HTML
- error message rendering request input
- preview endpoint reflecting form data

### Stored XSS
Attacker input is stored and later rendered to the same or other users.

Examples:
- comments
- profile fields
- CMS content
- support tickets
- chat messages
- admin dashboards rendering user content

### DOM XSS
Attacker input reaches a browser-side sink through JavaScript or client rendering logic.

Examples:
- `innerHTML`
- `outerHTML`
- `document.write`
- unsafe framework APIs that bypass normal escaping

---

# 2. Context Matters

Safe handling depends on **where** the data is rendered.

## 2.1 HTML text context
Example:
- inside a normal HTML element body

Typical control:
- HTML escaping

## 2.2 HTML attribute context
Example:
- inside `title=""`, `value=""`, `data-*=""`

Typical control:
- attribute-safe escaping
- quote-safe encoding
- avoid unsafe event handler attributes

## 2.3 JavaScript context
Example:
- inside `<script>`
- inside inline JS string
- script template generation

Typical control:
- avoid direct injection
- use safe serialization
- do not rely on plain HTML escaping

## 2.4 URL context
Example:
- `href`
- `src`
- redirect targets
- dynamic links

Typical control:
- scheme validation
- allowlisting
- context-aware encoding

## 2.5 HTML / raw markup context
Example:
- raw HTML rendering
- WYSIWYG content
- markdown rendered to HTML

Typical control:
- strict sanitization
- trusted content separation
- explicit safe-render policies

## 2.6 DOM insertion context
Example:
- `innerHTML`
- `outerHTML`
- fragment insertion
- unsafe client render helpers

Typical control:
- text insertion APIs
- safe DOM builders
- sanitizer before HTML insertion

**Important rule:** Escaping for one context is not automatically safe for another context.

---

# 3. Shared XSS Attack Surfaces

Prioritize these attack surfaces first:

- comments, chat, tickets, and messaging
- user profile fields and display names
- CMS, blog, article, and announcement content
- markdown and rich text editors
- preview pages
- search and reflected input pages
- admin dashboards that render user-generated content
- email/web preview generators
- notification content
- client-side DOM update helpers
- legacy template rendering paths
- alternate output formats for the same content

---

# 4. Shared Anti-Patterns

These are common danger signals across languages.

## A1. Unescaped template output
High-risk pattern:
- value inserted directly into template output without automatic escaping
- raw output tag
- explicit "safe" or "raw" render mode

Why risky:
Untrusted content may be interpreted by the browser as active markup or script.

## A2. Raw HTML rendering
High-risk pattern:
- raw HTML helper
- `safe` render flag
- bypassing autoescape
- rendering stored rich text directly

Why risky:
Raw HTML sinks require strict sanitization or trusted-only content.

## A3. Context mismatch
High-risk pattern:
- HTML escaping used inside JavaScript context
- plain string encoding used inside attribute or URL context
- no context-specific output handling

Why risky:
A value can be "escaped" but still exploitable in the wrong context.

## A4. Dangerous DOM sinks
High-risk pattern:
- `innerHTML`
- `outerHTML`
- `document.write`
- framework APIs that inject raw HTML

Why risky:
Client-side execution may occur even if the backend response is JSON rather than HTML.

## A5. Trusting markdown or rich text too much
High-risk pattern:
- markdown rendered directly to HTML
- WYSIWYG content shown without sanitizer
- preview and final render using different safety rules

Why risky:
Stored XSS often appears in content systems that allow formatting.

## A6. Sanitization assumed but not verified
High-risk pattern:
- variable named `sanitizedHtml`
- helper named `clean(...)`
- sanitizer configured elsewhere but not visible
- sanitizer applied in some views but not others

Why risky:
A naming convention does not prove safety.

## A7. Stored content later rendered in privileged views
High-risk pattern:
- user content later shown in admin panel, support dashboard, moderation view, export, or preview

Why risky:
Stored XSS can impact more privileged users than the attacker.

## A8. Client-side template misuse
High-risk pattern:
- API returns attacker-controlled content
- frontend inserts it into HTML sink
- server assumes JSON response means no XSS risk

Why risky:
The browser-side sink may still be dangerous.

---

# 5. Shared Protection Model

## 5.1 What strong protection usually looks like

Strong protections typically include:
- automatic escaping in templates
- context-appropriate output encoding
- safe text insertion APIs in the DOM
- strict HTML sanitization for rich content
- fixed allowlists for supported markup or attributes
- safe serialization into JavaScript contexts
- consistent protection across preview, storage, and final render paths

## 5.2 What does not automatically mean the code is safe

The following do **not** automatically mean the code is safe:
- using a template engine
- using a framework with autoescape support
- encoding in one place
- JSON response on the backend
- rich text feature with a sanitizer name present
- CSP being enabled
- content being "admin-only"
- markdown rendering existing

## 5.3 Autoescape is helpful but not universal
Autoescape may reduce basic HTML text-context XSS, but it does not automatically solve:
- raw render APIs
- HTML-safe bypass flags
- JavaScript context injection
- dangerous attributes
- URL-based execution contexts
- DOM XSS on the client side

---

# 6. False-Positive Controls

Do not report a vulnerability as `Confirmed` if:
- the content is provably constant or server-controlled,
- the value does not reach a browser-executable context,
- the framework's automatic escaping clearly applies and is not bypassed,
- a strong sanitizer is clearly applied before raw HTML rendering,
- the apparent sink is actually a safe text sink,
- the apparent dangerous API is used only with trusted fixed content.

Use `Suspected` or `Not enough evidence` if:
- the sink exists but the taint source is unclear,
- the source exists but the real sink is hidden behind a helper,
- sanitization or context-aware encoding may exist elsewhere but cannot be verified,
- stored content may be dangerous but the final render path is not visible.

Do not over-claim based only on:
- user-controlled input existing
- HTML-like strings in data
- presence of a rich text feature
- presence of a sanitizer name
- presence of frontend rendering without seeing the actual sink

---

# 7. Finding Classification

## Confirmed
Use `Confirmed` when there is clear evidence that:
- attacker-controlled input reaches a browser-interpreted sink,
- protection is absent, weak, bypassed, or context-inappropriate,
- and execution is plausibly possible.

## Suspected
Use `Suspected` when:
- there is a high-risk source-to-sink pattern,
- but sanitization, templating, or a hidden safety layer may still exist elsewhere.

## Not enough evidence
Use `Not enough evidence` when:
- the final rendering sink is not visible,
- the context cannot be determined,
- or framework behavior cannot be verified from the available code.

## Probably safe
Use `Probably safe` when:
- the path is visible,
- the sink uses safe escaping, safe text insertion, or strong sanitization,
- and no dangerous bypass or alternate unsafe render path is evident.

---

# 8. What Good Evidence Looks Like

Strong XSS findings usually include:
- the exact entry point
- the tainted input source
- the propagation path
- the rendering or execution sink
- the rendering context
- the missing or weak encoding / sanitization / safe-render control
- the reason attacker content can be interpreted by the browser

Good findings usually answer:
1. What input is attacker-controlled?
2. Where is it stored or reflected?
3. What sink renders it?
4. In what browser context is it interpreted?
5. What protection should have stopped execution?

---

# 9. Shared Remediation Guidance

Preferred fixes include:
- rely on safe template autoescaping where appropriate
- use context-appropriate output encoding
- replace raw HTML sinks with safe text sinks when possible
- strictly sanitize rich text or markdown before rendering
- keep trusted HTML and untrusted content clearly separated
- use safe serialization for JavaScript contexts
- validate or allowlist dangerous URL schemes and attributes
- apply the same protection across preview, storage, and final display

Avoid weak fixes such as:
- HTML escaping everywhere regardless of context
- blacklist filtering only
- trusting user roles instead of sanitizing output
- relying on CSP alone
- sanitizing only one display path while leaving another raw

---

# 10. Shared Quick Checklist

Use this as a reminder, not as a substitute for reasoning.

- Is there a real attacker-controlled source?
- Does it reach a template, HTML, DOM, or script sink?
- What rendering context is involved?
- Is the sink safe by default or explicitly unsafe?
- Is the applied protection appropriate for that context?
- Can stored user content later affect other users or admins?
- Are preview and final render paths equally protected?
- Is client-side rendering using safe text APIs or dangerous HTML sinks?
