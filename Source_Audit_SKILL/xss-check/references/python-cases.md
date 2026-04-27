# Python XSS Source Cases

## Purpose

This file contains Python-specific source point patterns and audit cases for XSS source discovery.

Use it when the target application is primarily implemented in Python, especially in:
- Django
- Django REST Framework with template rendering
- Flask
- FastAPI with server-rendered templates
- Jinja2-based applications
- Python backends exposing HTML views, admin panels, CMS features, rich-content rendering, or API-fed frontend content

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream rendering behavior in the target code.

---

# 1. Python Source Discovery Points

Prioritize these source values and events:
- Django/Flask/FastAPI route parameters, query values, form fields, JSON body fields, headers, cookies, uploaded filenames/metadata, and import rows
- values passed to Django templates, Jinja2 templates, `render`, `render_template`, `TemplateResponse`, HTML response constructors, or template context dictionaries
- comments, posts, markdown fields, profile fields, ticket content, chat messages, admin render data, notification content, CMS content, and stored HTML
- values marked with `|safe`, `mark_safe`, `Markup(...)`, custom safe filters, sanitizer wrappers, markdown renderers, or HTML-safe flags
- values inserted into script blocks, attributes, URL attributes, raw HTML snippets, email/web previews, and JSON APIs consumed by frontend rendering

Source questions:
- Which source supplies reflected input, stored content, template data, raw HTML, script data, attribute value, URL value, markdown, rich text, or API-delivered render content?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value escaped, sanitized, serialized, marked safe, converted from markdown, stored, or passed through a wrapper before rendering?
- Which rendering context should be audited next: HTML text, attribute, JavaScript, URL, raw HTML, rich text, template bypass, admin view, API-fed frontend, or alternate render path?

---

# 2. Python Source Patterns

## P-S1. Request-derived template source
Example idea:
- request value such as search text, profile input, preview body, or error message is added to a template context.

Audit relevance:
The source may render safely or unsafely depending on template expression and browser context.

Follow-up:
- inspect Django/Jinja/FastAPI template output, HTML responses, or frontend consumers.

## P-S2. Stored content display source
Example idea:
- comment, post body, CMS field, profile field, ticket content, or notification text is loaded and rendered.

Audit relevance:
Stored content can affect other users or admins depending on display paths.

Follow-up:
- identify writer path, user/admin render paths, and preview/final display consistency.

## P-S3. Safe, Markup, or trusted-content source
Example idea:
- values passed through `mark_safe`, `Markup`, custom safe filters, or sanitizer wrappers are later rendered as HTML.

Audit relevance:
The source may represent either a safe transformed value or untrusted content incorrectly marked safe.

Follow-up:
- inspect sanitizer configuration, trusted pipeline, and raw render destination.

## P-S4. Script, attribute, or URL context source
Example idea:
- context values are inserted into `<script>`, inline JavaScript, attributes, `href`, `src`, or template-generated URLs.

Audit relevance:
Context-specific handling matters; HTML text escaping may not be enough.

Follow-up:
- verify safe serialization, attribute escaping, and URL scheme restrictions.

## P-S5. Rich text, markdown, or preview source
Example idea:
- markdown, WYSIWYG content, or preview input is converted to HTML before display.

Audit relevance:
Conversion pipelines create source stages before and after sanitization.

Follow-up:
- trace conversion, sanitization, preview rendering, final rendering, and admin rendering.

## P-S6. API-fed frontend source
Example idea:
- Python backend returns JSON fields containing user content that frontend code later renders.

Audit relevance:
The backend source is still XSS-relevant if the frontend inserts it into raw HTML or script contexts.

Follow-up:
- inspect frontend consumers, especially raw HTML components and DOM insertion helpers.

---

# 3. Case Templates

## Case P-S-XSS-1: Template context source

Source focus:
Identify request or stored values passed to Django/Jinja/FastAPI template context.

Recommended follow-up:
Verify the final template context and whether escaping or safe/raw filters are used.

## Case P-S-XSS-2: Safe-marked source

Source focus:
Identify values passed through `|safe`, `mark_safe`, `Markup`, custom safe filters, or trusted HTML wrappers.

Recommended follow-up:
Verify trusted origin or strict sanitization before safe marking.

## Case P-S-XSS-3: Stored rich-content source

Source focus:
Identify stored markdown, HTML, CMS, comment, profile, or admin content later displayed in views.

Recommended follow-up:
Trace writer path and every display path, including preview and admin views.

## Case P-S-XSS-4: Script or URL context source

Source focus:
Identify template values inserted into scripts, event handlers, attributes, `href`, `src`, or JavaScript bootstrapping data.

Recommended follow-up:
Verify context-aware serialization and URL scheme controls.

---

# 4. Python-Specific Audit Heuristics

## 4.1 Django source heuristics
Pay attention to:
- request values passed to templates
- `render(...)` and `TemplateResponse`
- `|safe`
- custom template filters
- `mark_safe`
- admin views rendering user content
- stored comments/profile/CMS fields

## 4.2 Jinja and Flask source heuristics
Pay attention to:
- `render_template(...)` context values
- `|safe`
- `Markup(...)`
- manual HTML response assembly
- template values inside script or attribute contexts
- preview and rich-content routes

## 4.3 FastAPI source heuristics
Pay attention to:
- `Jinja2Templates`
- `HTMLResponse`
- template context dictionaries
- preview endpoints
- markdown or rich text rendering before template output

## 4.4 Rich-content source heuristics
Pay attention to:
- markdown renderers
- sanitizer wrappers
- HTML-safe flags
- `safeHtml` / `trustedHtml` values
- preview vs final render paths
- user view vs admin/moderation view

## 4.5 API-to-frontend source heuristics
Pay attention to:
- JSON fields carrying `html`, `content`, `body`, `message`, `description`, or `rendered` values
- DRF/FastAPI responses consumed by frontend components
- backend assumptions that JSON cannot create XSS

---

# 5. False-Positive Controls

Do not mark a Python source as high-priority if:
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

Good Python source evidence includes:
- route/view/worker/admin/import/preview entry point,
- source API such as request args, JSON body, form value, uploaded metadata, stored content, queue receive, or config record,
- propagation such as template context assignment, HTML construction, markdown conversion, sanitizer call, safe-wrapper marking, JSON serialization, or template variable passing,
- Django/Jinja/FastAPI template, HTML response, rich-content renderer, API response, or frontend consumer receiving the value,
- rendering context when visible.

Good source evidence answers:
1. Which Python entry point receives the rendering-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which template, raw HTML, script, attribute, URL, rich-content, API, frontend, admin, or alternate render behavior should be audited next?
4. Is the source used for reflected content, stored content, context data, raw HTML, safe HTML, markdown, script data, attribute value, URL value, or API-rendered content?

---

# 7. Quick Python Source Checklist

- Are request values passed to templates, HTML responses, scripts, attributes, or URLs?
- Are stored comments, profiles, tickets, CMS content, markdown, or rich text later rendered?
- Are values marked with `|safe`, `mark_safe`, `Markup`, or sanitizer wrappers passed toward raw render paths?
- Are preview, admin, email/web preview, export, and final display paths using the same source controls?
- Are backend JSON fields later rendered by frontend raw HTML or DOM sinks?
