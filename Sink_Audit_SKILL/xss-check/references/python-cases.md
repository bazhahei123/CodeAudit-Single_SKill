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

# 2. Python XSS Anti-Patterns

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

# 3. Case Templates

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

# 4. Python-Specific Audit Heuristics

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
