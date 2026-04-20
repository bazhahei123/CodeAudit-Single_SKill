# Python SSRF Cases

## Purpose

This file contains Python-specific SSRF patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Python, especially in:
- Django
- Flask
- FastAPI
- `requests`
- `httpx`
- `urllib`
- `aiohttp`
- Python backends exposing webhook tests, remote fetchers, previewers, importers, or generic HTTP helper services

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real request-target handling in the target code.

---

# 1. Python SSRF Control Points

When auditing Python applications, prioritize these control points.

## 1.1 Direct outbound request APIs
Look for:
- `requests.get/post/...`
- `httpx`
- `urllib`
- `aiohttp`
- helper wrappers around outbound requests

## 1.2 Request target construction
Look for:
- direct URL request parameters
- string URL concatenation
- callback URL storage and replay
- host/path recomposition
- import/preview target assembly

## 1.3 Redirect and client behavior
Look for:
- redirect-following defaults
- proxy behavior
- custom validator functions
- helper wrappers that hide real request handling

## 1.4 Indirect fetch paths
Look for:
- webhook test features
- URL preview helpers
- metadata fetchers
- remote file/image loaders
- queue jobs fetching stored URLs
- renderers importing remote resources

---

# 2. Python SSRF Anti-Patterns

### A1. Direct request to attacker-controlled URL
```python
return requests.get(url, timeout=5).text
```

Why risky:
User-controlled `url` may reach unintended internal or privileged destinations.

### A2. Weak scheme-only validation
```python
if url.startswith("http"):
    return requests.get(url)
```

Why risky:
A simple scheme check does not constrain the final destination.

### A3. Callback test endpoint with weak hostname check
```python
if "trusted.com" in callback_url:
    return httpx.get(callback_url)
```

Why risky:
String matching on hostnames is often insufficient.

### A4. Redirect-following request without final-destination checks
```python
return requests.get(url, allow_redirects=True)
```

Why risky:
A harmless-looking initial URL may redirect to an internal target.

### A5. Stored URL fetched later
```python
callback = webhook.target_url
return requests.post(callback, json=payload)
```

Why risky:
Stored URLs remain dangerous if attacker influence exists and no revalidation occurs.

---

# 3. Case Templates

## Case P-SSRF-1: Direct fetch SSRF

### Vulnerable pattern
```python
return requests.get(url).text
```

### Audit focus
Verify whether `url` is attacker-controlled and whether final destination constraints exist.

## Case P-SSRF-2: Weak callback validation

### Vulnerable pattern
```python
if "trusted.com" in callback_url:
    return httpx.get(callback_url)
```

### Audit focus
Verify whether hostname validation is strict and whether final resolution is checked.

## Case P-SSRF-3: Redirect-based bypass

### Vulnerable pattern
```python
return requests.get(url, allow_redirects=True)
```

### Audit focus
Verify whether redirect targets are revalidated.

## Case P-SSRF-4: Stored remote target

### Vulnerable pattern
```python
callback = webhook.target_url
return requests.post(callback, json=payload)
```

### Audit focus
Verify whether stored URLs are revalidated before use.

---

# 4. Python-Specific Audit Heuristics

## 4.1 Client API heuristics
Pay attention to:
- `requests`
- `httpx`
- `urllib`
- `aiohttp`
- helper wrappers around these clients

## 4.2 URL assembly heuristics
Pay attention to:
- string-built URLs
- partial host/path composition
- callback URL generation
- preview/import target construction
- helper methods named `fetch`, `download`, `preview`, or `verify`

## 4.3 Redirect and proxy heuristics
Pay attention to:
- `allow_redirects`
- session-wide client defaults
- proxy/no-proxy handling
- wrapper logic that silently follows redirects

## 4.4 Indirect fetch heuristics
Pay attention to:
- webhook testers
- URL previewers
- remote image/file import
- metadata/OpenGraph fetchers
- queue workers consuming stored URLs

## 4.5 Layer inconsistency heuristics
Check whether SSRF protection is consistent across:
- direct request path vs background job
- preview path vs webhook test path
- one request wrapper vs another
- initial URL validation vs final network call
