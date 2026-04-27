# PHP XSS Source Cases

## Purpose

This file contains PHP-specific source point patterns and audit cases for XSS source discovery.

Use it when the target application is primarily implemented in PHP, especially in:
- Laravel
- Symfony
- Twig
- Blade
- raw PHP templates
- PHP backends exposing HTML views, admin panels, CMS features, rich-content rendering, or API-fed frontend content

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream rendering behavior in the target code.

---

# 1. PHP Source Discovery Points

Prioritize these source values and events:
- `$_GET`, `$_POST`, `$_REQUEST`, route parameters, request input, headers, cookies, uploaded filenames/metadata, and import rows
- values passed to Blade, Twig, raw PHP templates, view data arrays, response builders, components, slots, or template partials
- comments, articles, profile fields, support tickets, chat messages, admin/moderation content, notification content, CMS content, markdown, rich text, and stored HTML
- values rendered by Blade `{{ }}`, Blade raw `{!! !!}`, Twig output, Twig `|raw`, direct `echo`, manual HTML construction, script blocks, attributes, URL attributes, and email/web previews
- values named or treated as `html`, `safeHtml`, `raw`, `trusted`, `bodyHtml`, `contentHtml`, or sanitizer output
- API fields that frontend code may later render as HTML or rich content

Source questions:
- Which source supplies reflected input, stored content, template data, raw HTML, script data, attribute value, URL value, markdown, rich text, or API-delivered render content?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value escaped, sanitized, serialized, marked safe, converted from markdown, stored, or passed through a wrapper before rendering?
- Which rendering context should be audited next: HTML text, attribute, JavaScript, URL, raw HTML, rich text, template bypass, admin view, API-fed frontend, or alternate render path?

---

# 2. PHP Source Patterns

## H-S1. Request-derived template source
Example idea:
- request value such as search text, profile input, preview body, or error message is passed to a view or echoed into a response.

Audit relevance:
The source may render safely or unsafely depending on the template expression and browser context.

Follow-up:
- inspect Blade, Twig, raw PHP template output, manual responses, or frontend consumers.

## H-S2. Stored content display source
Example idea:
- comment, article body, CMS field, profile field, ticket content, or notification text is loaded and rendered.

Audit relevance:
Stored content can affect other users or admins depending on display paths.

Follow-up:
- identify writer path, user/admin render paths, and preview/final display consistency.

## H-S3. Raw HTML or trusted-content source
Example idea:
- values named `bodyHtml`, `safeHtml`, `trustedHtml`, or sanitizer output are passed into Blade raw output, Twig `raw`, direct `echo`, or HTML helpers.

Audit relevance:
The source may represent either a safe transformed value or untrusted content incorrectly marked safe.

Follow-up:
- inspect sanitizer configuration, trusted pipeline, and raw render destination.

## H-S4. Script, attribute, or URL context source
Example idea:
- view data values are inserted into `<script>`, inline JavaScript, attributes, `href`, `src`, or template-generated URLs.

Audit relevance:
Context-specific handling matters; HTML text escaping may not be enough.

Follow-up:
- verify safe serialization, attribute escaping, and URL scheme restrictions.

## H-S5. Rich text, markdown, or preview source
Example idea:
- markdown, WYSIWYG content, or preview input is converted to HTML before display.

Audit relevance:
Conversion pipelines create source stages before and after sanitization.

Follow-up:
- trace conversion, sanitization, preview rendering, final rendering, and admin rendering.

## H-S6. API-fed frontend source
Example idea:
- PHP backend returns JSON fields containing user content that frontend code later renders.

Audit relevance:
The backend source is still XSS-relevant if the frontend inserts it into raw HTML or script contexts.

Follow-up:
- inspect frontend consumers, especially raw HTML components and DOM insertion helpers.

---

# 3. Case Templates

## Case H-S-XSS-1: View data source

Source focus:
Identify request or stored values passed to Blade, Twig, raw PHP templates, components, slots, or view data arrays.

Recommended follow-up:
Verify the final template context and whether escaped or raw output is used.

## Case H-S-XSS-2: Raw template source

Source focus:
Identify values passed toward Blade `{!! !!}`, Twig `|raw`, direct `echo`, or custom raw HTML helpers.

Recommended follow-up:
Verify trusted origin or strict sanitization before raw rendering.

## Case H-S-XSS-3: Stored rich-content source

Source focus:
Identify stored markdown, HTML, CMS, comment, profile, or admin content later displayed in views.

Recommended follow-up:
Trace writer path and every display path, including preview and admin views.

## Case H-S-XSS-4: Script or URL context source

Source focus:
Identify template values inserted into scripts, event handlers, attributes, `href`, `src`, or JavaScript bootstrapping data.

Recommended follow-up:
Verify context-aware serialization and URL scheme controls.

---

# 4. PHP-Specific Audit Heuristics

## 4.1 Laravel and Blade source heuristics
Pay attention to:
- `$request->input(...)`
- route parameters
- validation result arrays later passed to views
- Blade `{{ }}` and `{!! !!}`
- components rendering HTML props or slots
- preview and admin views
- inline script and attribute rendering

## 4.2 Symfony and Twig source heuristics
Pay attention to:
- request values passed to Twig
- `|raw`
- autoescape configuration
- custom filters marking content safe
- rich-content render paths
- admin/moderation templates

## 4.3 Raw PHP source heuristics
Pay attention to:
- `$_GET`, `$_POST`, and `$_REQUEST`
- direct `echo`
- mixed PHP/HTML templates
- manual response building
- request values reflected into pages
- stored content echoed from database fields

## 4.4 Rich-content source heuristics
Pay attention to:
- markdown renderers
- WYSIWYG HTML fields
- sanitizer wrappers
- `safeHtml` / `trustedHtml` values
- preview vs final render paths
- user view vs admin/moderation view

## 4.5 API-to-frontend source heuristics
Pay attention to:
- JSON fields carrying `html`, `content`, `body`, `message`, `description`, or `rendered` values
- API responses consumed by frontend components
- backend assumptions that JSON cannot create XSS

---

# 5. False-Positive Controls

Do not mark a PHP source as high-priority if:
- the value is fixed in trusted code,
- the value is only used in a safe text-only render path and no alternate render path is visible,
- the value never reaches templates, response construction, frontend rendering, rich-content conversion, raw HTML helpers, or browser-rendered output,
- the stored value is trusted-only and cannot be attacker-influenced.

Use `Suspected source` or `Not enough evidence` if:
- template behavior is hidden,
- sanitizer configuration is unavailable,
- frontend consumers are missing,
- stored writer paths are missing,
- raw vs escaped context is unclear.

---

# 6. What Good Evidence Looks Like

Good PHP source evidence includes:
- route/controller/script/worker/admin/import/preview entry point,
- source API such as request input, route value, uploaded metadata, stored content, queue receive, or config record,
- propagation such as view data assignment, HTML construction, markdown conversion, sanitizer call, safe-wrapper marking, JSON serialization, or template variable passing,
- Blade, Twig, raw PHP template, HTML response, rich-content renderer, API response, or frontend consumer receiving the value,
- rendering context when visible.

Good source evidence answers:
1. Which PHP entry point receives the rendering-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which template, raw HTML, script, attribute, URL, rich-content, API, frontend, admin, or alternate render behavior should be audited next?
4. Is the source used for reflected content, stored content, view data, raw HTML, safe HTML, markdown, script data, attribute value, URL value, or API-rendered content?

---

# 7. Quick PHP Source Checklist

- Are request values passed to templates, HTML responses, scripts, attributes, or URLs?
- Are stored comments, profiles, tickets, CMS content, markdown, or rich text later rendered?
- Are Blade raw output, Twig `raw`, direct `echo`, or sanitizer wrappers receiving dynamic values?
- Are preview, admin, email/web preview, export, and final display paths using the same source controls?
- Are backend JSON fields later rendered by frontend raw HTML or DOM sinks?
