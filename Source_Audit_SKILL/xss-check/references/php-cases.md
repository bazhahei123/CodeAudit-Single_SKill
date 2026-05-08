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

# 2. High-Coverage PHP XSS Source Candidate Inventory

Use these candidate lists to seed graph queries and text searches. Keep a candidate only when code shows browser-visible output, browser-interpreted HTML, template data, script data, attribute values, URL values, rich text, markdown, sanitizer input/output, trusted HTML wrappers, API content, or alternate render path relevance.

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

## 2.2 Queue, command, render, and admin entries

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
- preview controllers
- import controllers
- export controllers
- report controllers
- admin controllers
- moderation views
- email renderers
- notification renderers
- CMS controllers
- legacy admin scripts

## 2.3 Reflected and stored content source candidates

Search for request, DTO, array, model, form, job, or config fields named:
- `q`
- `query`
- `search`
- `keyword`
- `term`
- `message`
- `error`
- `reason`
- `title`
- `name`
- `display_name`
- `nickname`
- `username`
- `label`
- `description`
- `summary`
- `content`
- `body`
- `text`
- `comment`
- `reply`
- `review`
- `profile`
- `bio`
- `signature`
- `ticket`
- `notification`
- `announcement`
- `article`
- `post`
- `cms`

## 2.4 Template, response, and API propagation candidates

Search for source values passed into:
- Laravel `view(...)`
- `View::make`
- `compact(...)`
- `with(...)`
- Blade components
- Blade slots
- Symfony `render(...)`
- Twig context arrays
- raw PHP templates
- `echo`
- `print`
- `printf`
- `sprintf`
- `Response`
- `JsonResponse`
- API resource fields
- REST response arrays
- GraphQL response fields
- email template contexts
- report template contexts
- notification template contexts
- manual HTML string builders

## 2.5 Raw HTML, trusted wrapper, and context candidates

Search for:
- `html`
- `raw_html`
- `safe_html`
- `trusted_html`
- `body_html`
- `content_html`
- `message_html`
- `rendered`
- `markdown`
- `rich_text`
- `wysiwyg`
- `template`
- `script`
- `href`
- `src`
- `style`
- `onclick`
- Blade `{!!`
- Blade `!!}`
- Twig `|raw`
- `HtmlString`
- `Markup`
- custom safe filters
- sanitizer output

## 2.6 Rich text, markdown, sanitizer, and cross-layer candidates

Search for source values near:
- markdown renderer
- `league/commonmark`
- `Parsedown`
- `HTMLPurifier`
- `mewebstudio/Purifier`
- `sanitize`
- WYSIWYG editor content
- preview renderer
- final renderer
- admin renderer
- email/web preview renderer
- API field consumed by frontend
- frontend component prop
- hydrated JSON
- script data bootstrap

## 2.7 Downstream rendering relevance mapping candidates

After finding a source candidate, trace toward:
- Blade output
- Twig output
- raw PHP templates
- Blade `{!! !!}`
- Twig `|raw`
- direct `echo`
- response HTML construction
- markdown-to-HTML renderers
- sanitizer wrappers
- JSON API serializers
- frontend consumers
- admin/moderation views
- email/report renderers

## 2.8 PHP graph search recipes

Useful combinations:

```text
Route::get/Route::post + $request->input q/message + view/Blade context
$request->query/$_GET + error/search/title + echo/Response/template
API resource field html/body/content + frontend consumer/raw render
Laravel job/command + stored comment/notification + email/report renderer
markdown/Parsedown/CommonMark + stored body + Blade raw/Twig raw
HTMLPurifier/sanitize + safe_html/trusted_html + raw template output
sprintf/concat + request/stored field + text/html response
admin/moderation view + user content + {!! !!}/|raw/echo
```

---

# 3. PHP Source Patterns

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

# 4. Case Templates

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

# 5. PHP-Specific Audit Heuristics

## 5.1 Laravel and Blade source heuristics
Pay attention to:
- `$request->input(...)`
- route parameters
- validation result arrays later passed to views
- Blade `{{ }}` and `{!! !!}`
- components rendering HTML props or slots
- preview and admin views
- inline script and attribute rendering

## 5.2 Symfony and Twig source heuristics
Pay attention to:
- request values passed to Twig
- `|raw`
- autoescape configuration
- custom filters marking content safe
- rich-content render paths
- admin/moderation templates

## 5.3 Raw PHP source heuristics
Pay attention to:
- `$_GET`, `$_POST`, and `$_REQUEST`
- direct `echo`
- mixed PHP/HTML templates
- manual response building
- request values reflected into pages
- stored content echoed from database fields

## 5.4 Rich-content source heuristics
Pay attention to:
- markdown renderers
- WYSIWYG HTML fields
- sanitizer wrappers
- `safeHtml` / `trustedHtml` values
- preview vs final render paths
- user view vs admin/moderation view

## 5.5 API-to-frontend source heuristics
Pay attention to:
- JSON fields carrying `html`, `content`, `body`, `message`, `description`, or `rendered` values
- API responses consumed by frontend components
- backend assumptions that JSON cannot create XSS

---

# 6. False-Positive Controls

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

# 7. What Good Evidence Looks Like

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

# 8. Quick PHP Source Checklist

- Are request values passed to templates, HTML responses, scripts, attributes, or URLs?
- Are stored comments, profiles, tickets, CMS content, markdown, or rich text later rendered?
- Are Blade raw output, Twig `raw`, direct `echo`, or sanitizer wrappers receiving dynamic values?
- Are preview, admin, email/web preview, export, and final display paths using the same source controls?
- Are backend JSON fields later rendered by frontend raw HTML or DOM sinks?
