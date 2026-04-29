# nginx-misconfig-code-audit

This package contains a single `SKILL.md` for auditing Nginx misconfiguration issues directly from Nginx configuration and deployment files.

It covers:

- Missing root location
- Off-by-slash with `alias`
- Off-by-slash with `proxy_pass`
- Unsafe PHP FastCGI `SCRIPT_NAME`
- `$uri` / `$document_uri` CRLF injection in redirects
- Nginx variable expansion / SSI risk
- Raw backend response leakage
- `merge_slashes off`
