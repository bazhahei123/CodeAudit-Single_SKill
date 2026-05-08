# Python XSS Cases

## Purpose

This file contains Python-specific XSS patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Python, especially in:
- Django
- Django REST Framework with template rendering
- Flask
- FastAPI with server-rendered templates
- Jinja2-based applications
- Python backends exposing HTML views, admin panels, CMS features, or rich-content rendering

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real rendering sink in the target code.

---

# 1. Python XSS Control Points

When auditing Python applications, prioritize these control points.

## 1.1 Route and entry-point controls
Look for:
- Django views
- Flask routes
- FastAPI template routes
- preview endpoints
- CMS or content routes
- profile, comment, and messaging views

## 1.2 Template and render controls
Look for:
- Django templates
- Jinja2 templates
- autoescape behavior
- `|safe`
- `Markup(...)`
- manual HTML response construction
- values inserted into script blocks or attributes

## 1.3 Stored content paths
Look for:
- comments
- posts
- markdown fields
- profile fields
- tickets
- chat messages
- admin render views

## 1.4 Rich text / markdown controls
Look for:
- markdown renderers
- sanitizers
- preview rendering
- HTML-safe flags
- template filters that bypass escaping

---

# 2. High-Coverage Python XSS Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, rendering context, sink behavior, and missing context-appropriate controls.

## 2.1 Web, API, and request entry candidates
Search for:
- `@app.route`
- `@blueprint.route`
- `@bp.route`
- `Flask`
- `request.args`
- `request.form`
- `request.values`
- `request.get_json`
- `@api_view`
- `APIView`
- `ViewSet`
- `ModelViewSet`
- `GenericAPIView`
- `path(`
- `re_path(`
- `View`
- `dispatch`
- `get`
- `post`
- `@app.get`
- `@app.post`
- `FastAPI`
- `APIRouter`
- `Request`
- `GraphQLView`
- `Mutation`
- `admin`
- `preview`
- `comment`
- `profile`
- `message`

## 2.2 Template, response, and manual HTML sink candidates
Search for:
- `render`
- `render_template`
- `render_template_string`
- `TemplateResponse`
- `Jinja2Templates`
- `HTMLResponse`
- `HttpResponse`
- `StreamingHttpResponse`
- `mark_safe`
- `SafeString`
- `SafeText`
- `format_html` review
- `Markup`
- `MarkupSafe`
- `|safe`
- `autoescape false`
- `{% autoescape off %}`
- `{{`
- `return f"<`
- `return "<html`
- `Response(content=`
- `mimetype="text/html"`
- `content_type="text/html"`

## 2.3 Rich text, markdown, sanitizer, and script/URL candidates
Search for:
- `markdown.markdown`
- `Markdown(`
- `mistune`
- `markdown2`
- `bleach.clean`
- `bleach.linkify`
- `Cleaner`
- `nh3.clean`
- `html_sanitizer`
- `WYSIWYG`
- `body_html`
- `content_html`
- `json_script`
- `|tojson`
- `escapejs`
- `urlize`
- `href`
- `src`
- `javascript:`
- `script`
- `onclick`

## 2.4 Stored content and privileged render candidates
Search for:
- `Comment`
- `Post`
- `Article`
- `Profile`
- `Ticket`
- `Message`
- `Notification`
- `CMS`
- `admin.ModelAdmin`
- `list_display`
- `readonly_fields`
- `format_html`
- `moderation`
- `email preview`
- `report`
- `dashboard`

## 2.5 Required-control candidates
Search near sinks for:
- Jinja autoescape
- Django autoescape
- no `|safe`
- no `mark_safe`
- `escape`
- `conditional_escape`
- `format_html`
- `json_script`
- `|tojson`
- `escapejs`
- `bleach.clean`
- `nh3.clean`
- `Cleaner(tags=`
- allowed tags
- allowed attributes
- URL scheme allowlist
- `html.escape`
- `Content-Security-Policy`

## 2.6 Python graph search recipes
Useful combinations:

```text
@app.route + render_template_string
request.args + HTMLResponse
Django view + mark_safe
TemplateResponse + |safe
markdown.markdown + Markup
comment.body + |safe
admin view + user content + mark_safe
render_template + script block + user value
bleach.clean missing + rich text render
API response + frontend raw sink
```

---

# 3. Python XSS Anti-Patterns

### A1. Jinja / Django `safe` on untrusted content
```html
{{ comment.body|safe }}
```

Why risky:
`safe` disables escaping and allows raw HTML rendering.

### A2. Markup or raw HTML helper with user content
```python
return Markup(user_content)
```

Why risky:
This marks untrusted content as trusted HTML.

### A3. Manual HTML response construction
```python
return f"<div>{message}</div>"
```

Why risky:
User-controlled data may be reflected directly into HTML.

### A4. Data inserted into script context
```html
<script>
  const name = "{{ username }}";
</script>
```

Why risky:
Context must be verified; plain HTML-safe assumptions may not hold for script rendering.

---

# 4. Case Templates

## Case P-XSS-1: Unsafe template bypass

### Vulnerable pattern
```html
{{ post.content|safe }}
```

### Safer signal
Use default autoescaping for plain text, or sanitize rich content before intentional raw render.

## Case P-XSS-2: Manual HTML reflection

### Vulnerable pattern
```python
return f"<div>{q}</div>"
```

### Safer signal
Use template rendering with context-aware escaping instead of manual HTML assembly.

## Case P-XSS-3: Stored markdown rendering

### Vulnerable pattern
```python
html = markdown.markdown(post.body)
return render_template("post.html", body=Markup(html))
```

### Audit focus
Verify whether the markdown output is sanitized before being marked safe.

---

# 5. Python-Specific Audit Heuristics

## 4.1 Django heuristics
Pay attention to:
- autoescape behavior
- `|safe`
- custom template filters
- `mark_safe`
- admin views rendering user content

## 4.2 Jinja / Flask heuristics
Pay attention to:
- `|safe`
- `Markup(...)`
- manual HTML response assembly
- template values inside script or attribute contexts

## 4.3 FastAPI heuristics
Pay attention to:
- Jinja template use
- HTML responses
- preview endpoints
- markdown or rich text rendering before template output

## 4.4 Layer inconsistency heuristics
Check whether XSS protection is consistent across:
- preview vs final render
- API-fed frontend vs server-rendered views
- user view vs admin view
- plain text vs rich-content display
