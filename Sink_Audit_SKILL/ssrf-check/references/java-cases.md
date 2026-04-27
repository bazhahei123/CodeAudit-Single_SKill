# Java SSRF Cases

## Purpose

This file contains Java-specific SSRF patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- `RestTemplate`
- `WebClient`
- `HttpURLConnection`
- Apache HttpClient
- OkHttp
- Java backends exposing webhook tests, remote fetchers, previewers, importers, or generic HTTP helper services

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real request-target handling in the target code.

---

# 1. Java SSRF Control Points

When auditing Java applications, prioritize these control points.

## 1.1 Direct outbound request APIs
Look for:
- `RestTemplate`
- `WebClient`
- `HttpURLConnection`
- Apache HttpClient
- OkHttp
- wrapper helpers around network requests

## 1.2 Request target construction
Look for:
- `URI` / `URL` creation from request input
- string URL concatenation
- callback URL storage and replay
- import or preview URL assembly
- helper methods that split and recombine host, port, and path

## 1.3 Redirect and client behavior
Look for:
- client redirect configuration
- custom hostname verification logic
- proxy settings
- wrapper defaults that may expand protocol or redirect behavior

## 1.4 Indirect fetch paths
Look for:
- webhook test features
- remote image/file loaders
- HTML / metadata preview features
- queue jobs fetching stored URLs
- parser/render flows that load remote resources

---

# 2. Java SSRF Anti-Patterns

### A1. Direct request to attacker-controlled URL
```java
return restTemplate.getForObject(url, String.class);
```

Why risky:
User-controlled `url` may reach unintended internal or privileged destinations.

### A2. Weak URL filtering before request
```java
if (url.startsWith("http")) {
    return restTemplate.getForObject(url, String.class);
}
```

Why risky:
A scheme check alone does not constrain the final destination.

### A3. Callback verifier with weak host check
```java
if (callback.contains("trusted.com")) {
    client.send(HttpRequest.newBuilder(URI.create(callback)).build());
}
```

Why risky:
String matching on hostname is often insufficient.

### A4. Redirect-following client with initial validation only
```java
client = HttpClient.newBuilder().followRedirects(HttpClient.Redirect.ALWAYS).build();
```

Why risky:
A harmless-looking initial target may redirect to an internal destination.

### A5. Stored URL fetched later
```java
String callback = webhook.getTargetUrl();
return restTemplate.postForObject(callback, body, String.class);
```

Why risky:
Stored callback URLs remain dangerous if attacker influence exists and no revalidation occurs.

---

# 3. Case Templates

## Case J-SSRF-1: Direct fetcher SSRF

### Vulnerable pattern
```java
return restTemplate.getForObject(url, String.class);
```

### Audit focus
Verify whether `url` is attacker-controlled and whether final destination constraints exist.

## Case J-SSRF-2: Weak callback validation

### Vulnerable pattern
```java
if (callback.contains("trusted.com")) {
    client.send(...);
}
```

### Audit focus
Verify whether host validation is strict and whether final resolution is checked.

## Case J-SSRF-3: Redirect-based bypass

### Vulnerable pattern
```java
client = HttpClient.newBuilder().followRedirects(HttpClient.Redirect.ALWAYS).build();
```

### Audit focus
Verify whether redirect targets are revalidated.

## Case J-SSRF-4: Stored webhook target

### Vulnerable pattern
```java
String callback = webhook.getTargetUrl();
client.send(... callback ...);
```

### Audit focus
Verify whether stored URLs are revalidated before use.

---

# 4. Java-Specific Audit Heuristics

## 4.1 Client API heuristics
Pay attention to:
- `RestTemplate`
- `WebClient`
- `HttpURLConnection`
- Apache HttpClient
- OkHttp
- wrapper helpers around these clients

## 4.2 URI/URL heuristics
Pay attention to:
- `URI.create(...)`
- `new URL(...)`
- string-built URLs
- partial host/path recomposition
- callback and preview target assembly

## 4.3 Redirect and proxy heuristics
Pay attention to:
- automatic redirect configuration
- custom redirect handling
- proxy/no-proxy behavior
- wrapper defaults hidden in shared HTTP clients

## 4.4 Indirect fetch heuristics
Pay attention to:
- OpenGraph / metadata preview
- image/file remote loaders
- webhook test features
- import/render jobs
- queue workers using stored URLs

## 4.5 Layer inconsistency heuristics
Check whether SSRF protection is consistent across:
- direct fetchers vs webhook testers
- request path vs background job
- one client wrapper vs another
- initial validation vs final request path
