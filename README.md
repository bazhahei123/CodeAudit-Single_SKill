# AI Code Audit Source Skills Summary

## 1. Overall Architecture

```text
Source_Audit_SKILL/
├── access-control-check/
│   ├── SKILL.md
│   └── references/
│       ├── common-cases.md
│       ├── java-cases.md
│       ├── python-cases.md
│       └── php-cases.md
├── bussiness-logic-check/
│   ├── SKILL.md
│   └── references/
│       ├── authentication-cases.md
│       ├── common-cases.md
│       ├── payment-cases.md
│       ├── promotion-cases.md
│       ├── rate-limit-cases.md
│       ├── resource-consumption-cases.md
│       ├── third-party-integration-cases.md
│       └── workflow-cases.md
├── deserialization-check/
│   ├── SKILL.md
│   └── references/
│       ├── common-cases.md
│       ├── java-cases.md
│       ├── php-cases.md
│       └── python-cases.md
├── file-path-handling-check/
│   ├── SKILL.md
│   └── references/
│       ├── common-cases.md
│       ├── java-cases.md
│       ├── php-cases.md
│       └── python-cases.md
├── rce-check/
│   ├── SKILL.md
│   └── references/
│       ├── common-cases.md
│       ├── java-cases.md
│       ├── php-cases.md
│       └── python-cases.md
├── sql-injection-check/
│   ├── SKILL.md
│   └── references/
│       ├── common-cases.md
│       ├── java-sql-cases.md
│       ├── php-sql-cases.md
│       └── python-sql-cases.md
├── ssrf-check/
│   ├── SKILL.md
│   └── references/
│       ├── common-cases.md
│       ├── java-cases.md
│       ├── php-cases.md
│       └── python-cases.md
└── xss-check/
    ├── SKILL.md
    └── references/
        ├── common-cases.md
        ├── javascript-cases.md
        ├── java-cases.md
        ├── php-cases.md
        └── python-cases.md
```

## 2. Skill Coverage Summary

| Skill | Main Goal | Core Source Content |
|---|---|---|
| Access Control Source Check | Identify identity, role, object, tenant, relationship, and workflow-state sources that drive authorization decisions. | Principal sources, route/object IDs, tenant selectors, ownership attributes, delegated actors, policy context, client-supplied auth context. |
| Business Logic Source Check | Identify business-state, actor, amount, quota, workflow, callback, promotion, and integration inputs that can influence business rule execution. | Payment values, account binding, rate/quota keys, state transition inputs, promotion claims, resource usage requests, third-party events. |
| Deserialization Source Check | Identify serialized payload sources and type-selection inputs that may reach object restoration paths. | Request bodies, cookies, queues, cache records, files, session data, polymorphic type fields, framework binder inputs. |
| File Path Handling Source Check | Identify path, filename, archive entry, storage key, and resource locator sources that may influence file operations. | Upload names, download paths, template names, include paths, object keys, archive entries, symlink-adjacent paths. |
| RCE Source Check | Identify command, interpreter, script, template, plugin, expression, and external-tool inputs that may influence execution behavior. | Process arguments, shell fragments, eval expressions, template code, job payloads, environment/config values, tool options. |
| SQL Injection Source Check | Identify query-shaping values that may influence SQL text, structure, identifiers, filters, ordering, or ORM criteria. | Request filters, sort/order fields, dynamic table/column names, raw fragments, imported values, second-order query inputs. |
| SSRF Source Check | Identify URL, host, IP, callback, webhook, proxy, fetch, import, and redirect-controlled inputs that may influence outbound requests. | User URLs, avatar/image fetches, webhooks, integrations, preview/import jobs, metadata-like targets, DNS/redirect-influenced values. |
| XSS Source Check | Identify browser-rendering-relevant inputs that may flow toward server-rendered pages, DOM rendering, rich text, Markdown, or frontend raw rendering. | Reflected request values, stored content, template model values, API-to-frontend fields, browser-side sources, trusted/safe HTML markers. |

## 3. Usage Verification

### Access Control Source Check

/Evidence/access-control-audit-report-v2.md

![Access_proof](./Images/Access_proof.png)

---

## Need Your Help

Still developing，please give a **star**🌟 if it helps. Most of this project was completed with the help of AI, so  if it succeeds or fails, please file an issue for me.
