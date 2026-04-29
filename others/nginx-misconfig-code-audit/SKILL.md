# Nginx Misconfiguration Code Audit Skill

## Purpose

Audit Nginx configuration and deployment files for common security misconfigurations.

This skill is for white-box or gray-box code/configuration review. It focuses on identifying dangerous Nginx patterns directly from configuration files and providing the corresponding root cause, risk, and remediation in the same `SKILL.md`.

Covered cases:

1. Missing root location
2. Off-by-slash with `alias`
3. Off-by-slash with `proxy_pass`
4. Unsafe PHP FastCGI `SCRIPT_NAME`
5. `$uri` / `$document_uri` CRLF injection in redirects
6. Nginx variable expansion / SSI risk
7. Raw backend response leakage
8. `merge_slashes off`

---

## Files to Review

Search for Nginx configuration in:

```text
/etc/nginx/nginx.conf
/etc/nginx/conf.d/*.conf
/etc/nginx/sites-enabled/*
/etc/nginx/sites-available/*
nginx.conf
*.nginx.conf
Dockerfile
docker-compose.yml
kubernetes ingress YAML
Helm charts
Helm templates
deployment templates
reverse proxy templates
```

Also inspect application deployment files when they generate Nginx config dynamically.

---

## Core Search Keywords

Use these keywords to locate relevant configuration blocks:

```text
server {
location
root
alias
proxy_pass
fastcgi_pass
fastcgi_param
SCRIPT_FILENAME
SCRIPT_NAME
PATH_INFO
try_files
return 301
return 302
return 307
return 308
rewrite
$uri
$document_uri
$request_uri
ssi
merge_slashes
proxy_intercept_errors
proxy_hide_header
error_page
```

---

## General Audit Method

1. Identify every `server` block.
2. Identify all `location` blocks under each `server`.
3. Review `root`, `alias`, `proxy_pass`, `fastcgi_pass`, `return`, `rewrite`, `ssi`, `merge_slashes`, `proxy_intercept_errors`, and `proxy_hide_header`.
4. Check whether path boundaries are consistent.
5. Check whether user-controlled URI variables are used in headers, redirects, file paths, or FastCGI parameters.
6. Confirm whether a risky pattern is actually reachable.
7. Avoid reporting only by keyword. Include the vulnerable configuration snippet and explain the exact unsafe behavior.
8. Provide a concrete corrected configuration.

---

# Rule 1: Missing Root Location

## What to Check

Look for `root` defined at the `server` level, especially when the path points to a sensitive directory.

Risky examples:

```nginx
server {
    root /etc/nginx;

    location /hello.txt {
        try_files $uri $uri/ =404;
    }
}
```

```nginx
server {
    root /etc;

    location /static {
        try_files $uri =404;
    }
}
```

High-risk root paths:

```text
/etc
/etc/nginx
/var/log
/root
/home
/usr/local
/app/config
/opt/app/config
```

## Flag When

Flag as vulnerable or suspicious when:

```text
server-level root points to a sensitive directory
AND
there is no safe catch-all location /
```

Also flag when a server block exposes only specific locations but still has a broad global root.

## Why It Is Vulnerable

A globally scoped `root` may be used by Nginx to resolve unexpected request paths. If no safe `location /` exists, paths like `/nginx.conf` may map to local files such as `/etc/nginx/nginx.conf`.

## Risk

May expose:

```text
Nginx configuration
backend addresses
access logs
basic auth files
deployment paths
internal service routes
credentials or secret references
```

## False Positive Checks

Do not mark as confirmed if:

```text
root points to a dedicated static directory such as /var/www/html
AND
location / safely restricts access
AND
sensitive files are not reachable
```

Still mark as suspicious if the root path is sensitive, even if direct exposure is not proven.

## Fix

Use a dedicated web root:

```nginx
server {
    root /var/www/html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

If only specific files should be exposed, deny everything else:

```nginx
server {
    location / {
        return 404;
    }

    location = /hello.txt {
        root /var/www/html;
        try_files /hello.txt =404;
    }
}
```

Never use `/etc`, `/etc/nginx`, `/var/log`, `/root`, or application configuration directories as public web roots.

## Report Wording

The issue is caused by a globally scoped `root` directive pointing to a sensitive directory without a restrictive default `location /`. Nginx may resolve unexpected request paths against this root, which can expose local configuration files or other sensitive resources.

---

# Rule 2: Off-by-Slash with alias

## What to Check

Look for prefix `location` blocks where the location path does not end with `/`, but the `alias` path does end with `/`.

Risky example:

```nginx
location /static {
    alias /usr/share/nginx/static/;
}
```

Another risky example:

```nginx
location /assets {
    alias /app/public/assets/;
}
```

## Flag When

Flag when:

```text
location is a prefix location
AND
location path does not end with /
AND
alias target ends with /
```

Example pattern:

```text
location /static { alias /path/static/; }
```

## Why It Is Vulnerable

The `location` prefix does not enforce a clean directory boundary. A crafted request such as `/static../` may still match the `/static` location and be mapped unexpectedly. Depending on path normalization, this may allow access outside the intended static directory.

## Risk

May allow:

```text
static directory escape
source file disclosure
configuration file disclosure
access to files outside alias target
unexpected file mapping
```

## False Positive Checks

Lower severity if:

```text
the alias target contains only public static assets
AND
there are no sensitive files in adjacent parent directories
AND
additional access controls block traversal-like paths
```

Still recommend fixing trailing slash consistency.

## Fix

Use consistent trailing slashes:

```nginx
location /static/ {
    alias /usr/share/nginx/static/;
}
```

Optionally redirect the non-slash path:

```nginx
location = /static {
    return 301 /static/;
}
```

## Report Wording

The issue is caused by inconsistent path boundary handling between the Nginx `location` prefix and the `alias` target. Because the `location` path does not end with `/`, crafted paths may still match the prefix and be mapped outside the intended static file directory.

---

# Rule 3: Off-by-Slash with proxy_pass

## What to Check

Look for prefix `location` blocks where the location path does not end with `/`, while `proxy_pass` contains a URI path ending with `/`.

Risky example:

```nginx
location /api {
    proxy_pass http://apiserver/v1/;
}
```

Another risky example:

```nginx
location /backend {
    proxy_pass http://internal-service/;
}
```

## Flag When

Flag when:

```text
location is a prefix location
AND
location path does not end with /
AND
proxy_pass contains a URI path ending with /
```

Risk pattern:

```text
location /api
proxy_pass http://backend/v1/
```

## Why It Is Vulnerable

Nginx matches `/api` as a prefix. A crafted path such as `/api../` can still match this location. After prefix replacement, the backend may receive a path like `/v1/../`. If the backend normalizes that path, it may escape the intended `/v1/` prefix.

Example transformation:

```text
/api/user  -> http://apiserver/v1//user
/api../    -> http://apiserver/v1/../
```

The backend may normalize:

```text
/v1/../ -> /
```

## Risk

May expose backend-only paths such as:

```text
/server-status
/admin
/debug
/internal
/actuator
/metrics
```

This can lead to backend route bypass, debug endpoint exposure, or unintended access to internal APIs.

## False Positive Checks

Lower severity if:

```text
backend has its own strong authentication and path validation
AND
there are no sensitive routes outside the intended prefix
```

Still report the configuration weakness because Nginx path boundary is unsafe.

## Fix

Use consistent trailing slashes:

```nginx
location /api/ {
    proxy_pass http://apiserver/v1/;
}
```

Add exact redirect or deny rule:

```nginx
location = /api {
    return 301 /api/;
}
```

Do not rely only on Nginx prefix routing to protect backend internal paths.

## Report Wording

The issue is caused by inconsistent trailing slash usage between the Nginx `location` directive and `proxy_pass`. Since the `location` prefix does not enforce a strict directory boundary, crafted paths such as `/api../` may be normalized by the backend outside the intended route prefix.

---

# Rule 4: Unsafe PHP FastCGI SCRIPT_NAME

## What to Check

Look for PHP FastCGI blocks.

Risky examples:

```nginx
location ~ .php$ {
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_pass 127.0.0.1:9000;
}
```

```nginx
location ~ \.php$ {
    include fastcgi_params;
    fastcgi_pass php-fpm:9000;
}
```

High-risk sign: no `try_files $uri =404` before `fastcgi_pass`.

## Flag When

Flag when:

```text
fastcgi_pass exists
AND
PHP location handles .php requests
AND
there is no try_files $uri =404 before fastcgi_pass
```

Also flag broad PHP regex patterns:

```nginx
location ~ .php$
```

because `.` is not escaped. Prefer:

```nginx
location ~ \.php$
```

## Why It Is Vulnerable

Nginx may forward arbitrary URLs ending in `.php` to PHP-FPM even when the file does not exist. Attacker-controlled path content may enter FastCGI variables such as `SCRIPT_NAME`, `SCRIPT_FILENAME`, or `PATH_INFO`.

If the PHP application reflects these values, it may cause XSS, path confusion, or incorrect script execution behavior.

## Risk

May cause:

```text
reflected XSS
PATH_INFO confusion
unexpected PHP handling of non-existing files
information disclosure through PHP errors
application route bypass
```

## False Positive Checks

Do not mark as confirmed if:

```text
try_files $uri =404 exists before fastcgi_pass
AND
PHP-FPM only receives existing PHP files
```

If `fastcgi_split_path_info` is used, check whether the real script filename is validated.

## Fix

Require file existence before forwarding to PHP-FPM:

```nginx
location ~ \.php$ {
    try_files $uri =404;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_pass 127.0.0.1:9000;
}
```

If PATH_INFO is required, configure it carefully:

```nginx
location ~ ^(.+\.php)(/.+)$ {
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    try_files $fastcgi_script_name =404;

    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param PATH_INFO $fastcgi_path_info;
    fastcgi_pass 127.0.0.1:9000;
}
```

Application code should not reflect `SCRIPT_NAME`, `PATH_INFO`, or other server variables without output encoding.

## Report Wording

The issue is caused by forwarding PHP requests to PHP-FPM without verifying that the target PHP file exists. This allows user-controlled path segments to influence FastCGI variables such as `SCRIPT_NAME` or `PATH_INFO`, which may result in XSS, path confusion, or unexpected PHP request handling.

---

# Rule 5: $uri / $document_uri CRLF Injection in Redirects

## What to Check

Look for redirects using `$uri` or `$document_uri`.

Risky examples:

```nginx
location / {
    return 302 https://example.com$uri;
}
```

```nginx
rewrite ^ https://example.com$uri redirect;
```

```nginx
return 301 $scheme://$host$document_uri;
```

## Flag When

Flag when:

```text
return 301/302/307/308 uses $uri or $document_uri
OR
rewrite redirect uses $uri or $document_uri
```

Especially risky when user-controlled path data is placed into `Location` headers.

## Why It Is Vulnerable

`$uri` and `$document_uri` are normalized URI values. Nginx normalization may include URL decoding. Encoded CRLF sequences such as `%0d%0a` may become real line breaks when placed in a response header.

This can cause response header injection or HTTP response splitting.

## Risk

May lead to:

```text
response header injection
HTTP response splitting
cache poisoning
security header bypass
redirect manipulation
XSS in certain proxy/browser chains
```

## False Positive Checks

Lower severity if:

```text
the redirect target is fully fixed
AND
user-controlled URI data is not reflected into the Location header
AND
CRLF sequences are rejected before redirecting
```

## Fix

Prefer `$request_uri` when preserving the raw request URI:

```nginx
return 302 https://example.com$request_uri;
```

Reject encoded CRLF sequences:

```nginx
if ($request_uri ~* "%0d|%0a") {
    return 400;
}
```

Avoid inserting decoded URI variables into response headers.

Use fixed redirect paths where possible.

## Report Wording

The issue is caused by using normalized URI variables such as `$uri` or `$document_uri` inside redirect headers. Since these variables may contain decoded CRLF characters, attacker-controlled input can break the header boundary and inject additional response headers.

---

# Rule 6: Nginx Variable Expansion / SSI Risk

## What to Check

Look for SSI enabled in broad scopes:

```nginx
ssi on;
```

Risky examples:

```nginx
http {
    ssi on;
}
```

```nginx
server {
    ssi on;
}
```

```nginx
location / {
    ssi on;
}
```

Very risky if SSI is enabled on:

```text
user-uploaded content
dynamic content directories
public writable directories
template-rendered paths
```

## Flag When

Flag when:

```text
ssi on is enabled globally or broadly
OR
ssi on applies to user-controlled or uploaded content
OR
response content may contain user-controlled strings with Nginx variable syntax
```

## Why It Is Vulnerable

SSI or similar server-side processing may evaluate Nginx variables embedded in content. If user-controlled input reaches this evaluation context, values such as `$http_referer`, `$http_user_agent`, `$host`, or `$request_uri` may be expanded and reflected.

## Risk

May cause:

```text
information disclosure
request header reflection
server variable leakage
response content injection
possible XSS depending on rendering context
confusion between user input and server-side variable evaluation
```

## False Positive Checks

Do not mark as high risk if:

```text
SSI is enabled only for trusted static files
AND
users cannot modify SSI-processed content
AND
no request-derived values are reflected unsafely
```

Still document broad SSI usage as hardening concern.

## Fix

Disable SSI by default:

```nginx
ssi off;
```

Enable only for trusted static content:

```nginx
location /trusted-static/ {
    ssi on;
}
```

Never SSI-process user-uploaded or user-controlled content.

Encode any reflected request values.

## Report Wording

The issue is caused by enabling server-side variable evaluation in a context that may contain user-controlled input. If SSI or similar processing evaluates Nginx variables from untrusted content, attackers may cause variable disclosure or response content injection.

---

# Rule 7: Raw Backend Response Leakage

## What to Check

Look for configurations relying on Nginx to sanitize backend errors or headers:

```nginx
proxy_intercept_errors on;
proxy_hide_header Secret-Header;
error_page 500 /html/error.html;
```

This pattern is not automatically vulnerable, but it becomes risky when the backend generates sensitive headers or detailed error bodies.

## Flag When

Flag as suspicious when:

```text
proxy_intercept_errors on is used
OR
proxy_hide_header is used for sensitive headers
AND
backend may return sensitive debug data, secrets, stack traces, or internal headers
```

Also check whether backend debug mode is enabled in deployment files.

Search for sensitive backend headers:

```text
Secret-Header
X-Debug
X-Backend
X-Internal
X-Trace
X-Token
Authorization
Set-Cookie
```

## Why It Is Vulnerable

Nginx controls such as `proxy_intercept_errors` and `proxy_hide_header` work only when Nginx correctly parses the backend response. In abnormal request or protocol parsing cases, raw backend responses may be forwarded and sensitive data may bypass Nginx error interception or header hiding.

The safer design is that the backend should not generate sensitive error bodies or sensitive headers in the first place.

## Risk

May expose:

```text
backend error messages
debug output
stack traces
internal headers
framework information
service behavior
secrets accidentally included in error responses
```

## False Positive Checks

Do not mark as confirmed only because `proxy_intercept_errors` or `proxy_hide_header` exists.

Escalate severity if:

```text
backend code/config shows sensitive headers or debug errors
OR
production debug mode is enabled
OR
error responses include secrets or internal details
```

## Fix

Do not rely only on Nginx to hide sensitive backend data.

Backend-side fixes:

```text
disable debug mode in production
remove sensitive headers at the source
return generic error bodies
avoid including secrets in error messages
```

Nginx-side defense in depth:

```nginx
proxy_intercept_errors on;
proxy_hide_header X-Debug;
proxy_hide_header X-Internal;
error_page 500 502 503 504 /50x.html;
```

Also reject malformed requests where possible and keep Nginx and backend components updated.

## Report Wording

The issue is caused by relying on Nginx response interception and header hiding as the primary protection while the backend still generates sensitive error content or headers. In abnormal parsing cases, Nginx may fail to apply normal sanitization, causing raw backend information to be exposed.

---

# Rule 8: merge_slashes off

## What to Check

Look for:

```nginx
merge_slashes off;
```

It may appear in:

```nginx
http {
    merge_slashes off;
}
```

```nginx
server {
    merge_slashes off;
}
```

## Flag When

Flag when:

```text
merge_slashes off is configured
AND
Nginx acts as a reverse proxy
```

Increase severity if the backend has:

```text
path-based authorization
file download endpoints
static file serving
LFI risk
route restrictions
admin/internal routes
```

## Why It Is Vulnerable

By default, Nginx merges repeated slashes. When `merge_slashes off` is set, Nginx preserves repeated slashes, while backend frameworks may normalize them differently.

This creates inconsistent path interpretation between the edge proxy and backend application.

## Risk

May allow:

```text
path-based access control bypass
backend route confusion
LFI exploitation enhancement
static file boundary bypass
admin route bypass
WAF or routing rule bypass
```

## False Positive Checks

Lower severity if:

```text
backend also preserves repeated slashes consistently
AND
no path-based access control or file access is involved
AND
there is a documented business reason for disabling slash merging
```

Still recommend explicit normalization or rejection of repeated slashes.

## Fix

Keep default behavior:

```nginx
merge_slashes on;
```

Or remove the directive because `on` is the default.

Reject repeated slashes if not needed:

```nginx
if ($request_uri ~ "//+") {
    return 400;
}
```

Ensure backend and Nginx use consistent canonical path handling.

Do not base authorization decisions on uncanonicalized paths.

## Report Wording

The issue is caused by disabling Nginx's default repeated-slash normalization. This may create inconsistent path interpretation between Nginx and the backend, allowing route bypass, file path confusion, or access control bypass.

---

# Static Rule Summary

Use this section as the quick audit checklist.

| Case | Dangerous Pattern | Severity Guidance |
|---|---|---|
| Missing root location | `server`-level `root /etc/nginx` with no safe `location /` | High |
| Off-by-slash alias | `location /static` + `alias /path/static/` | High |
| Off-by-slash proxy | `location /api` + `proxy_pass http://backend/v1/` | High |
| Unsafe PHP FastCGI | `fastcgi_pass` without `try_files $uri =404` | Medium to High |
| `$uri` CRLF | `return 302 ...$uri` or `rewrite ...$uri redirect` | Medium to High |
| Variable expansion / SSI | broad `ssi on` | Medium |
| Raw backend response | relying on `proxy_hide_header` while backend returns sensitive errors | Medium to High |
| `merge_slashes off` | explicit `merge_slashes off` in reverse proxy | Medium to High |

---

# Suggested Semgrep-Like Pattern Ideas

These are not exact Semgrep rules, but they can guide automated scanning.

## Sensitive Root

```text
root /etc;
root /etc/nginx;
root /var/log;
root /root;
```

## Missing Default Location

```text
server block contains root
AND
server block does not contain location /
```

## Off-by-Slash alias

```text
location /NAME {
    alias /SOME/PATH/;
}
```

## Off-by-Slash proxy_pass

```text
location /NAME {
    proxy_pass http://HOST/PATH/;
}
```

## PHP FastCGI Without try_files

```text
location ~ \.php
AND
fastcgi_pass
AND NOT
try_files $uri =404
```

## `$uri` Redirect

```text
return 301 *$uri*
return 302 *$uri*
return 307 *$uri*
return 308 *$uri*
rewrite *$uri* redirect
```

## SSI Broadly Enabled

```text
ssi on;
```

## Raw Backend Response Risk

```text
proxy_intercept_errors on;
proxy_hide_header *
```

## Disabled Slash Normalization

```text
merge_slashes off;
```

---

# Audit Output Format

For every confirmed or suspicious finding, output the following:

## Finding

Nginx Misconfiguration - `<case name>`

## Severity

High / Medium / Low, based on reachability and exposed impact.

## Vulnerable Configuration

```nginx
<paste relevant config snippet>
```

## Why It Is Vulnerable

Explain the unsafe configuration behavior.

## Root Cause

State the exact configuration mistake.

## Impact

Describe what an attacker may access or bypass.

## False Positive Consideration

Mention whether the finding is confirmed or needs runtime validation.

## Remediation

Provide corrected Nginx configuration.

## Verification

Explain how to verify the fix through code review and, if applicable, interface testing.

---

# Example Finding: Off-by-Slash with proxy_pass

## Finding

Nginx Misconfiguration - Off-by-Slash in API Reverse Proxy

## Severity

High

## Vulnerable Configuration

```nginx
location /api {
    proxy_pass http://apiserver/v1/;
}
```

## Why It Is Vulnerable

The `location` prefix `/api` does not end with `/`, while the `proxy_pass` target path `/v1/` does. This creates an ambiguous path boundary. A crafted request such as `/api../` may be forwarded as `/v1/../`, allowing the backend to normalize the path outside the intended `/v1/` prefix.

## Root Cause

The root cause is inconsistent trailing slash usage between the Nginx `location` directive and `proxy_pass`.

## Impact

This may expose backend-only routes, debug endpoints, server-status pages, or administrative interfaces.

## False Positive Consideration

If the backend enforces authentication and has no sensitive routes outside `/v1/`, practical impact may be reduced. However, the Nginx routing boundary is still unsafe and should be corrected.

## Remediation

```nginx
location /api/ {
    proxy_pass http://apiserver/v1/;
}

location = /api {
    return 301 /api/;
}
```

## Verification

After fixing, confirm that the configuration uses consistent path boundaries. Runtime validation can verify that `/api../`, `/apiuser`, and `/api../server-status` do not reach backend-only paths.
```
