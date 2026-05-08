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

# 2. High-Coverage Python SSRF Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, final request target, sink behavior, and missing destination controls.

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
- `request.cookies`
- `request.headers`
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
- `Query`
- `Body`
- `Request`
- `GraphQLView`
- `Mutation`

## 2.2 Worker, message, admin, preview, and import entries
Search for:
- `@shared_task`
- `@app.task`
- `Celery`
- `dramatiq.actor`
- `rq.job`
- `BaseCommand`
- `handle`
- `KafkaConsumer`
- `basic_consume`
- `on_message`
- `webhook`
- `callback`
- `preview`
- `fetch`
- `import`
- `download`
- `avatar`
- `image`
- `feed`
- `metadata`
- `crawler`
- `screenshot`
- `admin`
- `diagnostic`
- `replay`

## 2.3 Request target construction candidates
Search for:
- `url`
- `uri`
- `host`
- `hostname`
- `endpoint`
- `callback_url`
- `webhook_url`
- `image_url`
- `feed_url`
- `metadata_url`
- `preview_url`
- `base_url`
- `endpoint_url`
- `urljoin`
- `urlparse`
- `urllib.parse`
- `unquote`
- `socket.gethostbyname`
- `socket.getaddrinfo`
- `ipaddress.ip_address`
- `base64.b64decode`
- `redirect`
- `proxy`

## 2.4 Python HTTP, URL, socket, and async sink candidates
Search for:
- `requests.get`
- `requests.post`
- `requests.put`
- `requests.request`
- `requests.Session`
- `session.get`
- `session.post`
- `httpx.get`
- `httpx.post`
- `httpx.request`
- `httpx.Client`
- `httpx.AsyncClient`
- `urllib.request.urlopen`
- `urllib.request.Request`
- `aiohttp.ClientSession`
- `session.request`
- `session.get`
- `tornado.httpclient.AsyncHTTPClient`
- `treq.get`
- `pycurl`
- `socket.create_connection`
- `socket.connect`
- `ftplib.FTP`
- `smtplib.SMTP`

## 2.5 Parser, renderer, cloud SDK, and indirect fetch sink candidates
Search for:
- `BeautifulSoup`
- `feedparser.parse`
- `PIL.Image.open`
- `lxml.etree.parse`
- `xml.etree.ElementTree.parse`
- `weasyprint.HTML`
- `pdfkit.from_url`
- `imgkit.from_url`
- `selenium.webdriver`
- `driver.get`
- `playwright`
- `page.goto`
- `boto3.client`
- `endpoint_url`
- `botocore.config.Config`
- `OpenGraph`
- `metadata fetch`
- `screenshot`
- `crawler`

## 2.6 Redirect, DNS, proxy, and protocol candidates
Search for:
- `allow_redirects`
- `follow_redirects`
- `max_redirects`
- `proxies`
- `trust_env`
- `NO_PROXY`
- `HTTP_PROXY`
- `HTTPS_PROXY`
- `verify=False`
- `socket.getaddrinfo`
- `gethostbyname`
- `ipaddress`
- `file://`
- `ftp://`
- `gopher://`
- `dict://`

## 2.7 Required-control candidates
Search near sinks for:
- `allowlist`
- `allowed_hosts`
- `allowed_schemes`
- `urlparse(url).scheme`
- `urlparse(url).hostname`
- `ipaddress.ip_address`
- `is_private`
- `is_loopback`
- `is_link_local`
- `is_multicast`
- `169.254.169.254`
- `metadata.google.internal`
- `localhost`
- `127.0.0.1`
- `::1`
- `allow_redirects=False`
- redirect revalidation
- final IP check
- DNS pinning
- `timeout=`
- `stream=True` review

## 2.8 Python graph search recipes
Useful combinations:

```text
@app.route + requests.get
request.args + httpx.get
FastAPI + urllib.request.urlopen
APIView + callback_url + requests.post
@shared_task + stored URL + httpx
allow_redirects=True + user URL
urljoin + request parameter + requests
boto3.client + endpoint_url
weasyprint.HTML + user URL
driver.get + preview URL
```

---

# 3. Python SSRF Anti-Patterns

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

# 4. Case Templates

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

# 5. Python-Specific Audit Heuristics

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
