# Java XSS Cases

## Purpose

This file contains Java-specific XSS patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- JSP
- Thymeleaf
- FreeMarker
- Servlet-based web applications
- Java backends exposing HTML views, admin panels, CMS features, or rich-content rendering

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real rendering sink in the target code.

---

# 1. Java XSS Control Points

When auditing Java applications, prioritize these control points.

## 1.1 Route and entry-point controls
Look for:
- `@Controller`
- `@RestController`
- `Model`
- `ModelAndView`
- servlet handlers
- view-returning controllers
- preview endpoints
- rich content management routes

## 1.2 Template and render controls
Look for:
- JSP output
- Thymeleaf expressions
- `th:text`
- `th:utext`
- FreeMarker output
- manual HTML string construction
- values inserted into script blocks or attributes

## 1.3 Stored content paths
Look for:
- comments
- articles
- CMS fields
- profile fields
- ticket content
- notification content
- admin moderation views

## 1.4 Rich text / markdown controls
Look for:
- markdown renderers
- WYSIWYG content fields
- HTML sanitizers
- preview rendering before storage
- different render paths for admin vs user views

---

# 2. Java XSS Anti-Patterns

### A1. Thymeleaf raw output with untrusted data
```html
<div th:utext="${comment.content}"></div>
```

Why risky:
`th:utext` renders unescaped HTML.

### A2. JSP direct unescaped output
```jsp
<%= request.getParameter("q") %>
```

Why risky:
User-controlled input is inserted directly into HTML output.

### A3. Manual HTML construction in controller or service
```java
String html = "<div>" + message + "</div>";
model.addAttribute("content", html);
```

Why risky:
Unsafe string assembly may bypass template protections.

### A4. Data inserted into script context
```html
<script>
  var name = '[[${username}]]';
</script>
```

Why risky:
Context handling must be verified carefully; plain HTML escaping is not enough for every script use case.

---

# 3. Case Templates

## Case J-XSS-1: Unescaped JSP output

### Vulnerable pattern
```jsp
<%= request.getParameter("q") %>
```

### Safer signal
Use context-appropriate escaping or safe template output conventions instead of direct raw insertion.

## Case J-XSS-2: Raw Thymeleaf rendering

### Vulnerable pattern
```html
<div th:utext="${post.body}"></div>
```

### Safer signal
Use `th:text` for plain text, or sanitize rich content before using raw HTML rendering.

## Case J-XSS-3: Stored rich-content rendering

### Vulnerable pattern
```java
model.addAttribute("content", article.getBodyHtml());
```

### Audit focus
Verify whether `getBodyHtml()` is trusted, sanitized, and consistently safe across preview and final rendering.

---

# 4. Java-Specific Audit Heuristics

## 4.1 JSP heuristics
Pay attention to:
- `<%= ... %>`
- request parameter reflection
- manual output helpers
- raw script or attribute insertion

## 4.2 Thymeleaf heuristics
Pay attention to:
- `th:text` vs `th:utext`
- inline JavaScript rendering
- attribute generation
- preview or CMS views

## 4.3 FreeMarker heuristics
Pay attention to:
- autoescape settings
- no-escape directives
- raw HTML model attributes
- custom HTML helpers

## 4.4 Layer inconsistency heuristics
Check whether XSS protection is consistent across:
- preview vs final render
- user view vs admin view
- list view vs detail view
- HTML output vs email/web preview
