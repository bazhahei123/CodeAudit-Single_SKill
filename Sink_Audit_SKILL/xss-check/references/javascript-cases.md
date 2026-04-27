# JavaScript / TypeScript XSS Cases

## Purpose

This file contains JavaScript- and TypeScript-specific XSS patterns, anti-patterns, and audit cases.

Use it when the target application includes:
- browser-side rendering
- DOM manipulation
- React / Vue / Angular / Svelte / Next.js / Nuxt frontend code
- client-side markdown or rich-text rendering
- SPA code that receives backend-controlled content
- admin panels or dashboards that render user-generated content in the browser

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow, real browser sink, real rendering context, and any sanitizer or framework protection in the target code.

---

# 1. JavaScript / TypeScript XSS Control Points

When auditing frontend or browser-side code, prioritize these control points.

## 1.1 DOM sinks
Look for:
- `innerHTML`
- `outerHTML`
- `insertAdjacentHTML`
- `document.write`
- `Range.createContextualFragment`
- manual HTML string building before DOM insertion
- wrapper helpers that eventually write HTML into the DOM

Questions:
- Does attacker-controlled content reach an HTML-interpreting DOM sink?
- Is the sink used with backend data, query parameters, hash fragments, postMessage data, stored content, or rich text?

## 1.2 Framework-specific raw rendering
Look for:
- React `dangerouslySetInnerHTML`
- Vue `v-html`
- Angular bypass helpers such as `bypassSecurityTrustHtml`
- Svelte `{@html ...}`
- framework wrappers around these features
- custom “safeHtml” or “renderHtml” helpers

Questions:
- Is raw HTML rendering truly necessary?
- Is the content sanitized before reaching the raw render sink?
- Is the sanitizer strict and consistently applied?

## 1.3 Client-side source handling
Look for:
- `location.search`
- `location.hash`
- `document.URL`
- `window.name`
- `postMessage`
- localStorage / sessionStorage
- API responses containing user-generated content
- stored profile/comment/post/message fields
- rich text or markdown payloads

Questions:
- What attacker-controlled data sources are used by the client?
- Are they inserted into DOM sinks, framework raw render sinks, or script-relevant contexts?

## 1.4 Rich text / markdown / sanitizer paths
Look for:
- markdown renderers
- WYSIWYG editors
- HTML sanitizers such as DOMPurify
- custom sanitizer wrappers
- preview rendering
- client-side transforms from markdown to HTML

Questions:
- Is markdown or rich text converted into HTML in the browser?
- Is sanitization applied before render?
- Is the sanitizer configured safely and consistently?

## 1.5 Script and URL contexts
Look for:
- inline script construction
- event handler attribute generation
- dynamic `href`, `src`, or URL injection
- `javascript:` scheme handling
- client-side template string injection into script blocks or inline handlers

Questions:
- Is user input reaching script, handler, or URL-based execution contexts?
- Is context-appropriate handling applied?

---

# 2. JavaScript / TypeScript XSS Anti-Patterns

These are high-risk signals. They are not automatic proof of a vulnerability, but they should trigger deeper review.

## 2.1 Dangerous DOM sink anti-patterns

### A1. `innerHTML` with attacker-controlled content
```javascript
element.innerHTML = userContent;
```

Why risky:
The browser interprets `userContent` as HTML and potentially executable markup.

What to verify:
- Is `userContent` derived from query parameters, API data, stored content, or message events?
- Is there strong sanitization immediately before assignment?

### A2. `insertAdjacentHTML` with dynamic content
```javascript
container.insertAdjacentHTML("beforeend", html);
```

Why risky:
This is another raw HTML sink and should be treated like `innerHTML`.

What to verify:
- Is `html` attacker-influenced?
- Is the inserted content sanitized or trusted?

### A3. `document.write` or equivalent
```javascript
document.write(message);
```

Why risky:
`document.write` is an HTML-interpreting sink and often unsafe with untrusted data.

What to verify:
- Does `message` come from attacker-controlled input?
- Can the sink be reached in modern rendering or preview paths?

---

## 2.2 Framework raw-render anti-patterns

### A4. React `dangerouslySetInnerHTML`
```jsx
<div dangerouslySetInnerHTML={{ __html: content }} />
```

Why risky:
This explicitly bypasses React's normal text escaping.

What to verify:
- Is `content` sanitized before render?
- Is the sanitizer strict enough for the actual content type?
- Is the same content ever rendered through an unsafe alternate path?

### A5. Vue `v-html`
```vue
<div v-html="content"></div>
```

Why risky:
`v-html` renders raw HTML and bypasses normal template escaping.

What to verify:
- Is `content` attacker-controlled?
- Is there a sanitizer or allowlist-based content pipeline?

### A6. Angular security bypass helpers
```typescript
this.safeHtml = this.sanitizer.bypassSecurityTrustHtml(content);
```

Why risky:
Security bypass helpers disable framework protections and must be treated as highly sensitive.

What to verify:
- Why is bypass needed?
- Is `content` actually trusted or sanitized?
- Is the bypass wrapper reused too broadly?

### A7. Svelte raw HTML
```svelte
{@html content}
```

Why risky:
This renders raw HTML and should be treated like any other raw sink.

What to verify:
- Is `content` attacker-controlled?
- Is it sanitized before render?

---

## 2.3 Rich content and markdown anti-patterns

### A8. Markdown rendered directly to HTML and inserted
```javascript
const html = marked.parse(markdown);
preview.innerHTML = html;
```

Why risky:
Markdown renderers can produce HTML, and insertion into an HTML sink may create XSS if sanitization is missing or weak.

What to verify:
- Is raw HTML allowed in markdown?
- Is HTML sanitized after markdown conversion?
- Are preview and final render paths consistent?

### A9. Trusting sanitizer by name only
```javascript
const safe = sanitizeHtml(content);
element.innerHTML = safe;
```

Why risky:
A helper name alone does not prove safety.

What to verify:
- What sanitizer is actually used?
- How is it configured?
- Is the same sanitizer used on every equivalent path?

---

## 2.4 Client-side source-to-sink anti-patterns

### A10. URL fragment or query data into DOM sink
```javascript
const q = new URLSearchParams(location.search).get("q");
result.innerHTML = q;
```

Why risky:
Reflected or DOM-based XSS can occur if request-derived browser data reaches raw sinks.

What to verify:
- Is the data inserted as HTML or text?
- Is there proper escaping or a safe text sink?

### A11. `postMessage` data into HTML sink
```javascript
window.addEventListener("message", (event) => {
  panel.innerHTML = event.data;
});
```

Why risky:
`postMessage` can carry attacker-controlled content from another origin or context.

What to verify:
- Is origin validated?
- Is `event.data` sanitized before HTML insertion?

### A12. API content to raw render sink
```javascript
fetch("/api/post/1")
  .then(r => r.json())
  .then(data => {
    container.innerHTML = data.body;
  });
```

Why risky:
The backend may store attacker-controlled content that becomes stored XSS in the frontend.

What to verify:
- Is `data.body` user-generated?
- Is there sanitization before render?
- Are there admin or privileged viewer paths?

---

# 3. Case Templates

Use these as reasoning patterns, not as direct proof.

## Case JS-XSS-1: DOM sink XSS

### Vulnerable pattern
```javascript
element.innerHTML = userContent;
```

### Why risky
The browser interprets `userContent` as HTML rather than plain text.

### What to verify
- Is `userContent` attacker-controlled?
- Is sanitization present and correctly configured?
- Would a text sink such as `textContent` be the appropriate alternative?

### Safer pattern
```javascript
element.textContent = userContent;
```

---

## Case JS-XSS-2: React raw HTML render

### Vulnerable pattern
```jsx
<div dangerouslySetInnerHTML={{ __html: content }} />
```

### Why risky
This bypasses React's normal escaping behavior.

### What to verify
- Is `content` trusted or sanitized?
- Is the sanitizer strict and consistently applied?
- Is there any alternate path where the same content is rendered raw without controls?

### Safer signals
- plain JSX text rendering
- sanitized trusted HTML pipeline
- no raw HTML sink for attacker-controlled content

---

## Case JS-XSS-3: Vue raw HTML render

### Vulnerable pattern
```vue
<div v-html="content"></div>
```

### Why risky
This bypasses Vue's normal escaping and renders raw HTML.

### What to verify
- Is `content` attacker-controlled?
- Is a sanitizer applied before render?

### Safer signals
- normal template interpolation
- sanitized trusted HTML pipeline
- no unsafe fallback render path

---

## Case JS-XSS-4: Markdown preview XSS

### Vulnerable pattern
```javascript
const html = marked.parse(markdown);
preview.innerHTML = html;
```

### Why risky
Markdown conversion may produce unsafe HTML, especially if raw HTML is allowed or sanitizer configuration is weak.

### What to verify
- Is HTML allowed in markdown?
- Is there post-render sanitization?
- Are preview and persisted render paths aligned?

### Safer signals
- raw HTML disabled in markdown pipeline
- strong sanitizer after markdown conversion
- no raw DOM insertion without sanitization

---

## Case JS-XSS-5: postMessage to DOM sink

### Vulnerable pattern
```javascript
window.addEventListener("message", (event) => {
  container.innerHTML = event.data;
});
```

### Why risky
Cross-context attacker-controlled data reaches a raw HTML sink.

### What to verify
- Is origin validated?
- Is the payload sanitized?
- Is the sink necessary at all?

### Safer signals
- strict origin allowlist
- text sinks instead of HTML sinks
- structured payload validation and safe rendering

---

# 4. JavaScript / TypeScript-Specific Audit Heuristics

## 4.1 DOM API heuristics
Pay attention to:
- `innerHTML`
- `outerHTML`
- `insertAdjacentHTML`
- `document.write`
- `createContextualFragment`
- wrapper utilities that eventually call these APIs

Audit questions:
- Is the sink HTML-interpreting?
- Could a safe text sink be used instead?
- Is there a sanitizer immediately and reliably in front of the sink?

## 4.2 React heuristics
Pay attention to:
- `dangerouslySetInnerHTML`
- wrappers around raw HTML components
- markdown or rich text components
- API-provided HTML content
- admin preview/render components

Audit questions:
- Is the HTML pipeline trusted and sanitized?
- Are the same props ever passed from untrusted backend content?

## 4.3 Vue heuristics
Pay attention to:
- `v-html`
- custom directives rendering HTML
- CMS or preview components
- stored content display
- wrapper components that forward raw HTML props

Audit questions:
- Is `v-html` used only for trusted content?
- Is the data sanitized consistently?

## 4.4 Angular heuristics
Pay attention to:
- `bypassSecurityTrustHtml`
- `bypassSecurityTrustUrl`
- `bypassSecurityTrustResourceUrl`
- dynamic template-like behavior
- custom sanitizer wrappers

Audit questions:
- Why is the security bypass needed?
- Is the trusted boundary real and narrow?

## 4.5 Rich text / markdown heuristics
Pay attention to:
- DOMPurify or alternative sanitizers
- markdown pipelines
- preview renderers
- editor plugins
- custom HTML allowlists

Audit questions:
- Is sanitization after HTML generation?
- Are preview, edit, and final display paths aligned?
- Is the allowlist too broad for the application's risk level?

## 4.6 Browser data-source heuristics
Pay attention to:
- `location.search`
- `location.hash`
- `postMessage`
- localStorage/sessionStorage
- backend JSON rendered into DOM
- websocket/event-stream content

Audit questions:
- Can the attacker influence this browser-side source?
- Does it eventually reach an HTML, script, URL, or handler sink?

## 4.7 Layer inconsistency heuristics
Check whether XSS protection is consistent across:
- backend-rendered page vs SPA component
- preview vs final render
- normal user view vs admin view
- markdown path vs stored rich-content path
- one frontend framework component vs another wrapper path

Common failure:
One component sanitizes or renders safely, but a second preview/admin/helper component uses a raw sink with the same content.

---

# 5. False-Positive Controls

Do not report a vulnerability as confirmed if:
- the content is provably constant or trusted-only,
- the apparent sink is actually a safe text sink,
- the framework's default escaping clearly applies and is not bypassed,
- a strong sanitizer is clearly applied before every equivalent raw render path,
- the dangerous API is used only with fixed trusted markup.

If sanitization, origin validation, or framework-level protection may exist elsewhere but cannot be confirmed:
- mark the result as `Suspected`, or
- mark the result as `Not enough evidence`.

Do not over-claim based only on:
- use of a frontend framework,
- presence of `innerHTML` in dead or fixed-content code,
- presence of a sanitizer name,
- existence of rich text or markdown support,
- receipt of backend content without showing the actual sink.

---

# 6. What Good Evidence Looks Like

Strong frontend XSS findings usually include:
- the exact browser-side source
- the propagation path
- the DOM or framework sink
- the rendering context
- the missing or weak sanitizer / safe-render control
- the reason the browser interprets the content as active markup or script

Good findings usually answer:
1. What input is attacker-controlled?
2. Where does the frontend receive it?
3. What sink renders it?
4. What context is involved?
5. What protection should have stopped execution?

---

# 7. Quick JavaScript / TypeScript XSS Audit Checklist

Use this as a reminder, not as a substitute for reasoning.

- Does attacker-controlled data reach `innerHTML`, `outerHTML`, `insertAdjacentHTML`, or `document.write`?
- Does React use `dangerouslySetInnerHTML` with untrusted content?
- Does Vue use `v-html` with untrusted content?
- Does Angular use security-bypass helpers?
- Does markdown or rich text become HTML before sanitization?
- Do query parameters, hash fragments, API responses, or postMessage payloads reach HTML sinks?
- Are preview and final render paths equally protected?
- Would a text sink be safer than the current raw HTML sink?
