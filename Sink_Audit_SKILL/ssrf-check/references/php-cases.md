# PHP SSRF Cases

## Purpose

This file contains PHP-specific SSRF patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in PHP, especially in:
- Laravel
- Symfony
- raw PHP applications
- `curl`
- `file_get_contents` with remote URLs
- Guzzle
- PHP backends exposing webhook tests, remote fetchers, previewers, importers, or generic HTTP helper services

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real request-target handling in the target code.

---

# 1. PHP SSRF Control Points

When auditing PHP applications, prioritize these control points.

## 1.1 Direct outbound request APIs
Look for:
- cURL usage
- `file_get_contents` on URLs
- Guzzle clients
- framework HTTP clients
- helper wrappers around outbound requests

## 1.2 Request target construction
Look for:
- direct URL request parameters
- string URL concatenation
- stored callback URL usage
- host/path recomposition
- import/preview target assembly

## 1.3 Redirect and client behavior
Look for:
- follow-redirect options
- protocol support options
- proxy behavior
- custom validator helpers
- wrapper defaults

## 1.4 Indirect fetch paths
Look for:
- webhook test features
- URL preview helpers
- remote download/import features
- metadata fetchers
- queue jobs using stored URLs
- renderers that load remote resources

---

# 2. PHP SSRF Anti-Patterns

### A1. Direct request to attacker-controlled URL
```php
$ch = curl_init($url);
curl_exec($ch);
```

Why risky:
User-controlled `$url` may reach unintended internal or privileged destinations.

### A2. Weak scheme-only validation
```php
if (strpos($url, 'http') === 0) {
    echo file_get_contents($url);
}
```

Why risky:
A simple scheme check does not constrain the final destination.

### A3. Weak callback hostname check
```php
if (strpos($callbackUrl, 'trusted.com') !== false) {
    $client->request('POST', $callbackUrl);
}
```

Why risky:
String matching on hostnames is often insufficient.

### A4. Redirect-following request without revalidation
```php
curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
```

Why risky:
A harmless-looking initial URL may redirect to an internal target.

### A5. Stored URL fetched later
```php
$callback = $webhook->target_url;
$client->request('POST', $callback, ['json' => $payload]);
```

Why risky:
Stored URLs remain dangerous if attacker influence exists and no revalidation occurs.

---

# 3. Case Templates

## Case H-SSRF-1: Direct fetch SSRF

### Vulnerable pattern
```php
$ch = curl_init($url);
curl_exec($ch);
```

### Audit focus
Verify whether `$url` is attacker-controlled and whether final destination constraints exist.

## Case H-SSRF-2: Weak callback validation

### Vulnerable pattern
```php
if (strpos($callbackUrl, 'trusted.com') !== false) {
    $client->request('POST', $callbackUrl);
}
```

### Audit focus
Verify whether hostname validation is strict and whether final resolution is checked.

## Case H-SSRF-3: Redirect-based bypass

### Vulnerable pattern
```php
curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
```

### Audit focus
Verify whether redirect targets are revalidated.

## Case H-SSRF-4: Stored remote target

### Vulnerable pattern
```php
$callback = $webhook->target_url;
$client->request('POST', $callback, ['json' => $payload]);
```

### Audit focus
Verify whether stored URLs are revalidated before use.

---

# 4. PHP-Specific Audit Heuristics

## 4.1 Client API heuristics
Pay attention to:
- cURL
- `file_get_contents` with URLs
- Guzzle
- framework HTTP clients
- helper wrappers around these clients

## 4.2 URL assembly heuristics
Pay attention to:
- string-built URLs
- host/path recomposition
- callback URL handling
- preview/import target assembly
- wrapper helpers around outbound fetch

## 4.3 Redirect and protocol heuristics
Pay attention to:
- `CURLOPT_FOLLOWLOCATION`
- protocol restrictions
- stream wrapper support
- proxy/no-proxy handling
- helper defaults that expand allowed transports

## 4.4 Indirect fetch heuristics
Pay attention to:
- webhook test paths
- previewers
- remote file/image loaders
- metadata/OpenGraph fetchers
- queue jobs using stored URLs

## 4.5 Layer inconsistency heuristics
Check whether SSRF protection is consistent across:
- direct fetch path vs webhook test path
- one HTTP wrapper vs another
- preview/import path vs normal path
- initial validation vs final network call
