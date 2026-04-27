# Java XSS Source Cases

## Purpose

This file contains Java-specific source point patterns and audit cases for XSS source discovery.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- JSP
- Thymeleaf
- FreeMarker
- Servlet-based web applications
- Java backends exposing HTML views, admin panels, CMS features, rich-content rendering, or API-fed frontend content

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream rendering behavior in the target code.

---

# 1. Java Source Discovery Points

Prioritize these source values and events:
- `@PathVariable`, `@RequestParam`, `@RequestBody`, headers, cookies, multipart filenames/metadata, and import DTO fields
- values added to `Model`, `ModelMap`, `ModelAndView`, request attributes, session attributes, servlet responses, or view context objects
- comments, articles, CMS fields, profile fields, ticket content, notification content, admin moderation content, markdown, rich text, and stored HTML
- values rendered by JSP, Thymeleaf, FreeMarker, server-side HTML builders, script blocks, attributes, URL attributes, and email/web previews
- values named or treated as `html`, `safeHtml`, `raw`, `trusted`, `bodyHtml`, `contentHtml`, or sanitizer output
- API fields that frontend code may later render as HTML or rich content

Source questions:
- Which source supplies reflected input, stored content, template model data, raw HTML, script data, attribute value, URL value, markdown, rich text, or API-delivered render content?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value escaped, sanitized, serialized, marked safe, converted from markdown, stored, or passed through a wrapper before rendering?
- Which rendering context should be audited next: HTML text, attribute, JavaScript, URL, raw HTML, rich text, template bypass, admin view, API-fed frontend, or alternate render path?

---

# 2. Java Source Patterns

## J-S1. Request-derived template source
Example idea:
- request value such as search text, profile input, preview body, or error message is added to a view model.

Audit relevance:
The source may render safely or unsafely depending on the template expression and browser context.

Follow-up:
- inspect JSP, Thymeleaf, FreeMarker, servlet response, or frontend component output.

## J-S2. Stored content display source
Example idea:
- comment, article body, CMS field, profile field, ticket content, or notification text is loaded and rendered.

Audit relevance:
Stored content can affect other users or admins depending on display paths.

Follow-up:
- identify writer path, user/admin render paths, and preview/final display consistency.

## J-S3. Raw HTML or trusted-content source
Example idea:
- values named `bodyHtml`, `safeHtml`, `trustedHtml`, or sanitizer output are passed into raw template constructs or HTML helpers.

Audit relevance:
The source may represent either a safe transformed value or untrusted content incorrectly marked safe.

Follow-up:
- inspect sanitizer configuration, trusted pipeline, and raw render destination.

## J-S4. Script, attribute, or URL context source
Example idea:
- model values are inserted into `<script>`, inline JavaScript, attributes, `href`, `src`, or template-generated URLs.

Audit relevance:
Context-specific handling matters; HTML text escaping may not be enough.

Follow-up:
- verify safe serialization, attribute escaping, and URL scheme restrictions.

## J-S5. Rich text, markdown, or preview source
Example idea:
- markdown, WYSIWYG content, or preview input is converted to HTML before display.

Audit relevance:
Conversion pipelines create source stages before and after sanitization.

Follow-up:
- trace conversion, sanitization, preview rendering, final rendering, and admin rendering.

## J-S6. API-fed frontend source
Example idea:
- Java backend returns JSON fields containing user content that frontend code later renders.

Audit relevance:
The backend source is still XSS-relevant if the frontend inserts it into raw HTML or script contexts.

Follow-up:
- inspect frontend consumers, especially raw HTML components and DOM insertion helpers.

---

# 3. Case Templates

## Case J-S-XSS-1: Model attribute source

Source focus:
Identify request or stored values added to `Model`, `ModelAndView`, or servlet request attributes.

Recommended follow-up:
Verify the final template context and whether escaping or raw output is used.

## Case J-S-XSS-2: Raw template source

Source focus:
Identify values passed toward JSP raw output, Thymeleaf `th:utext`, FreeMarker no-escape output, or custom raw HTML helpers.

Recommended follow-up:
Verify trusted origin or strict sanitization before raw rendering.

## Case J-S-XSS-3: Stored rich-content source

Source focus:
Identify stored markdown, HTML, CMS, comment, profile, or admin content later displayed in views.

Recommended follow-up:
Trace writer path and every display path, including preview and admin views.

## Case J-S-XSS-4: Script or URL context source

Source focus:
Identify template model values inserted into scripts, event handlers, attributes, `href`, `src`, or JavaScript bootstrapping data.

Recommended follow-up:
Verify context-aware serialization and URL scheme controls.

---

# 4. Java-Specific Audit Heuristics

## 4.1 Spring and servlet source heuristics
Pay attention to:
- `@Controller` and view-returning routes
- `@RequestParam`, `@PathVariable`, and `@RequestBody`
- `Model.addAttribute(...)`
- `ModelAndView.addObject(...)`
- servlet `request.setAttribute(...)`
- manual `HttpServletResponse` writing
- preview, CMS, profile, comment, ticket, and admin routes

## 4.2 JSP source heuristics
Pay attention to:
- request parameters or attributes used in JSP
- `<%= ... %>` output
- scriptlet-built HTML
- raw script or attribute insertion
- values passed into custom JSP tags

## 4.3 Thymeleaf and FreeMarker source heuristics
Pay attention to:
- `th:text` vs `th:utext`
- inline JavaScript rendering
- attribute generation
- FreeMarker autoescape settings
- no-escape directives
- preview or CMS views

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
- REST/GraphQL responses consumed by frontend components
- backend assumptions that JSON cannot create XSS

---

# 5. False-Positive Controls

Do not mark a Java source as high-priority if:
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

Good Java source evidence includes:
- route/controller/worker/admin/import/preview entry point,
- source annotation or API such as request param, DTO field, stored content, queue receive, or config record,
- propagation such as model assignment, HTML construction, markdown conversion, sanitizer call, safe-wrapper marking, JSON serialization, or template variable passing,
- JSP, Thymeleaf, FreeMarker, servlet response, rich-content renderer, API response, or frontend consumer receiving the value,
- rendering context when visible.

Good source evidence answers:
1. Which Java entry point receives the rendering-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which template, raw HTML, script, attribute, URL, rich-content, API, frontend, admin, or alternate render behavior should be audited next?
4. Is the source used for reflected content, stored content, model data, raw HTML, safe HTML, markdown, script data, attribute value, URL value, or API-rendered content?

---

# 7. Quick Java Source Checklist

- Are request values added to templates, views, HTML responses, scripts, attributes, or URLs?
- Are stored comments, profiles, tickets, CMS content, markdown, or rich text later rendered?
- Are values named `html`, `safeHtml`, `trustedHtml`, `bodyHtml`, or `contentHtml` passed toward raw render paths?
- Are preview, admin, email/web preview, export, and final display paths using the same source controls?
- Are backend JSON fields later rendered by frontend raw HTML or DOM sinks?
