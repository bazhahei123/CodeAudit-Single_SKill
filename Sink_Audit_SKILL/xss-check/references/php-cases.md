# PHP XSS Cases

## Purpose

This file contains PHP-specific XSS patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in PHP, especially in:
- Laravel
- Symfony
- Twig
- Blade
- raw PHP templates
- PHP backends exposing HTML views, admin panels, CMS features, or rich-content rendering

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real rendering sink in the target code.

---

# 1. PHP XSS Control Points

When auditing PHP applications, prioritize these control points.

## 1.1 Route and entry-point controls
Look for:
- controller methods
- Blade/Twig render paths
- raw PHP template files
- preview endpoints
- CMS routes
- profile, comment, and messaging views

## 1.2 Template and render controls
Look for:
- Blade `{{ }}` vs `{!! !!}`
- Twig escaped output vs `|raw`
- direct `echo`
- manual HTML response construction
- values inserted into script blocks or attributes

## 1.3 Stored content paths
Look for:
- comments
- articles
- profile fields
- support tickets
- chat messages
- admin and moderation views

## 1.4 Rich text / markdown controls
Look for:
- markdown renderers
- WYSIWYG content
- sanitizers
- preview rendering
- raw render helpers

---

# 2. PHP XSS Anti-Patterns

### A1. Blade raw output
```php
{!! $comment->body !!}
```

Why risky:
Raw Blade output bypasses escaping.

### A2. Twig raw filter
```twig
{{ post.body|raw }}
```

Why risky:
The `raw` filter disables escaping and renders HTML directly.

### A3. Direct echo of user input
```php
echo $_GET['q'];
```

Why risky:
User-controlled input may be reflected directly into HTML.

### A4. Manual HTML concatenation
```php
$html = "<div>" . $message . "</div>";
echo $html;
```

Why risky:
Unsafe string assembly may bypass template protections.

---

# 3. Case Templates

## Case H-XSS-1: Raw Blade rendering

### Vulnerable pattern
```php
{!! $post->content !!}
```

### Safer signal
Use escaped output for plain text, or sanitize rich content before intentional raw render.

## Case H-XSS-2: Twig raw rendering

### Vulnerable pattern
```twig
{{ comment.body|raw }}
```

### Safer signal
Keep autoescaping enabled unless sanitized trusted HTML is required.

## Case H-XSS-3: Raw PHP reflection

### Vulnerable pattern
```php
echo $_GET['q'];
```

### Safer signal
Use context-appropriate escaping instead of direct raw output.

---

# 4. PHP-Specific Audit Heuristics

## 4.1 Blade heuristics
Pay attention to:
- `{{ }}` vs `{!! !!}`
- components rendering HTML props
- preview and admin views
- inline script and attribute rendering

## 4.2 Twig heuristics
Pay attention to:
- `|raw`
- autoescape configuration
- custom filters marking content safe
- rich-content render paths

## 4.3 Raw PHP heuristics
Pay attention to:
- direct `echo`
- mixed PHP/HTML templates
- manual response building
- request values reflected into pages

## 4.4 Layer inconsistency heuristics
Check whether XSS protection is consistent across:
- preview vs final render
- user view vs admin view
- template engine vs raw PHP fallback
- plain text vs rich-content display
