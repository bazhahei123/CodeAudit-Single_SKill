# JavaScript / TypeScript XSS Source Cases

## Purpose

This file contains JavaScript- and TypeScript-specific source point patterns and audit cases for XSS source discovery.

Use it when the target application includes:
- browser-side rendering
- DOM manipulation
- React / Vue / Angular / Svelte / Next.js / Nuxt frontend code
- client-side markdown or rich-text rendering
- SPA code that receives backend-controlled content
- admin panels or dashboards that render user-generated content in the browser

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, downstream browser rendering behavior, and any sanitizer or framework protection in the target code.

---

# 1. JavaScript / TypeScript Source Discovery Points

Prioritize these source values and events:
- `location.search`, `location.hash`, `document.URL`, `document.referrer`, `window.name`, route params, and query params
- `postMessage` payloads, localStorage, sessionStorage, cookies, WebSocket messages, EventSource messages, and BroadcastChannel messages
- API responses containing user-generated content, rich text, markdown, sanitized HTML, trusted HTML, labels, descriptions, or notification text
- React props, Vue props, Angular component inputs, Svelte props, frontend store values, router state, and hydrated server data
- values passed toward `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write`, `createContextualFragment`, and raw DOM wrapper helpers
- values passed toward `dangerouslySetInnerHTML`, `v-html`, Angular bypass helpers, Svelte `{@html ...}`, custom raw HTML components, markdown renderers, or sanitizer wrappers
- values named or treated as `html`, `safeHtml`, `trustedHtml`, `rawHtml`, `bodyHtml`, `contentHtml`, `markdown`, `messageHtml`, or sanitizer output

Source questions:
- Which source supplies browser-side input, API content, component prop, DOM data, markdown, rich text, safe HTML, trusted HTML, or raw HTML?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, server-trusted, mixed, or unclear?
- Is the value parsed, sanitized, transformed, marked trusted, stored in frontend state, or passed through a wrapper before rendering?
- Which browser context should be audited next: text rendering, HTML insertion, framework raw render, script, URL, attribute, rich-text preview, final display, admin view, or alternate component path?

---

# 2. JavaScript / TypeScript Source Patterns

## JS-S1. Browser URL or storage source
Example idea:
- `location.search`, `location.hash`, router params, localStorage, or sessionStorage value is read and later displayed.

Audit relevance:
Browser-side sources can create DOM XSS if they reach HTML-interpreting sinks.

Follow-up:
- trace into text rendering, DOM insertion, router-rendered components, and framework raw render features.

## JS-S2. Cross-context message source
Example idea:
- `postMessage`, WebSocket, EventSource, or BroadcastChannel payload is displayed in a panel, notification, preview, or log view.

Audit relevance:
External contexts can supply attacker-controlled data unless origin and payload are constrained.

Follow-up:
- verify origin checks, payload validation, and downstream render context.

## JS-S3. API content source
Example idea:
- backend JSON field such as `body`, `description`, `content`, `html`, or `message` flows into frontend rendering.

Audit relevance:
Stored or reflected backend content can become XSS-relevant at the frontend sink.

Follow-up:
- trace API response to props, state, store, raw render components, markdown renderers, and DOM sinks.

## JS-S4. Raw HTML component prop source
Example idea:
- a prop named `html`, `dangerousHtml`, `safeHtml`, or `contentHtml` is passed into React, Vue, Angular, Svelte, or custom raw render components.

Audit relevance:
The prop is a high-priority source even if the component name implies safety.

Follow-up:
- inspect sanitizer origin, trusted boundary, and every raw render use.

## JS-S5. Markdown, rich text, or sanitizer pipeline source
Example idea:
- markdown or rich text is converted to HTML, sanitized, stored in state, and rendered in preview or final display.

Audit relevance:
Source stages before and after sanitization should be distinguished.

Follow-up:
- verify sanitizer configuration, raw HTML allowance, preview/final consistency, and alternate render paths.

## JS-S6. Framework bypass or trusted-value source
Example idea:
- Angular `bypassSecurityTrust*`, React `dangerouslySetInnerHTML`, Vue `v-html`, Svelte `{@html ...}`, or a custom trusted wrapper receives a dynamic value.

Audit relevance:
Framework bypasses mark values as safe or raw and should be traced back to true origin.

Follow-up:
- verify trusted origin, sanitizer placement, and scope of bypass wrapper reuse.

---

# 3. Case Templates

## Case JS-S-XSS-1: Browser-side source

Source focus:
Identify values from URL, storage, messaging, or browser APIs that later affect rendering.

Recommended follow-up:
Verify whether they reach safe text rendering or HTML/script/URL/attribute sinks.

## Case JS-S-XSS-2: API content source

Source focus:
Identify API response fields carrying user-generated text, HTML, markdown, or rich content.

Recommended follow-up:
Trace frontend consumers and verify actual rendering context.

## Case JS-S-XSS-3: Raw component prop source

Source focus:
Identify component props or state values passed to raw HTML render features.

Recommended follow-up:
Verify sanitizer pipeline and whether the prop is attacker-influenced.

## Case JS-S-XSS-4: Markdown preview source

Source focus:
Identify markdown/rich-text source values before conversion, after conversion, and after sanitization.

Recommended follow-up:
Verify conversion and sanitization are consistent across preview and final rendering.

## Case JS-S-XSS-5: Cross-context message source

Source focus:
Identify `postMessage`, WebSocket, EventSource, or external widget payloads that enter UI rendering.

Recommended follow-up:
Verify origin validation, schema validation, and safe rendering.

---

# 4. JavaScript / TypeScript-Specific Audit Heuristics

## 4.1 Browser data-source heuristics
Pay attention to:
- `location.search`
- `location.hash`
- `document.URL`
- `document.referrer`
- `window.name`
- router params
- query params
- localStorage/sessionStorage

## 4.2 Message and async source heuristics
Pay attention to:
- `postMessage`
- WebSocket data
- EventSource data
- BroadcastChannel data
- third-party widget events
- notification/event payloads

## 4.3 API and state source heuristics
Pay attention to:
- backend JSON rendered into UI
- fields named `html`, `body`, `content`, `message`, `description`, `rendered`, `safeHtml`, or `trustedHtml`
- Redux/Vuex/Pinia/Zustand/state values
- hydrated server-side data
- admin dashboard data

## 4.4 DOM and raw render source heuristics
Pay attention to values passed toward:
- `innerHTML`
- `outerHTML`
- `insertAdjacentHTML`
- `document.write`
- `Range.createContextualFragment`
- wrapper utilities that eventually write HTML into the DOM

## 4.5 Framework source heuristics
Pay attention to values passed toward:
- React `dangerouslySetInnerHTML`
- Vue `v-html`
- Angular `bypassSecurityTrustHtml`
- Angular `bypassSecurityTrustUrl`
- Angular `bypassSecurityTrustResourceUrl`
- Svelte `{@html ...}`
- raw HTML wrapper components
- custom directives rendering HTML

## 4.6 Rich text and sanitizer source heuristics
Pay attention to:
- DOMPurify or alternative sanitizer inputs and outputs
- markdown pipelines
- WYSIWYG editor content
- preview renderers
- custom HTML allowlists
- values marked trusted after sanitization

## 4.7 Layer inconsistency source heuristics
Check whether source handling is consistent across:
- backend-rendered page vs SPA component
- preview vs final render
- normal user view vs admin view
- markdown path vs stored rich-content path
- one frontend framework component vs another wrapper path
- API display path vs email/web preview path

---

# 5. False-Positive Controls

Do not mark a JavaScript/TypeScript source as high-priority if:
- the value is fixed trusted markup,
- the value only reaches safe text rendering and no alternate raw render path is visible,
- the value never reaches DOM rendering, framework rendering, raw HTML components, rich-text conversion, sanitizer output, or browser-interpreted contexts,
- the source cannot be influenced by an attacker or weakly trusted producer.

Use `Suspected source` or `Not enough evidence` if:
- component internals are hidden,
- sanitizer configuration is unavailable,
- API field writer paths are missing,
- message origin validation is unclear,
- the actual DOM sink is hidden behind a wrapper.

---

# 6. What Good Evidence Looks Like

Good JavaScript/TypeScript source evidence includes:
- browser-side source, API response, component input, frontend store, or external message entry point,
- source API such as URL APIs, storage APIs, `postMessage`, WebSocket/EventSource, fetch response, prop, state, or sanitizer output,
- propagation such as parsing, state assignment, prop passing, markdown conversion, sanitizer call, trusted wrapper construction, or DOM helper call,
- DOM API, framework raw render feature, component, markdown renderer, sanitizer wrapper, or browser context receiving the value,
- rendering context when visible.

Good source evidence answers:
1. Which frontend or API entry point receives the rendering-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, server-trusted, mixed, or unclear?
3. Which DOM, framework, rich-text, sanitizer, script, URL, attribute, preview, final display, admin, or alternate component behavior should be audited next?
4. Is the source used for browser URL data, external message data, API content, component prop, state value, raw HTML, safe HTML, markdown, rich text, or sanitizer output?

---

# 7. Quick JavaScript / TypeScript Source Checklist

- Are browser URL, storage, message, WebSocket, or EventSource values rendered?
- Are API fields containing user-generated content passed to frontend components?
- Are values named `html`, `safeHtml`, `trustedHtml`, `bodyHtml`, or `contentHtml` passed to raw render features?
- Are markdown/rich-text values converted and rendered consistently across preview and final display?
- Are framework bypass helpers receiving dynamic values?
- Are admin dashboards and alternate components rendering the same content through different source paths?
