# AI Code Audit Skills Summary

## 1. Overall Architecture

```text
skills/
├── access-control-check/
│   ├── SKILL.md
│   └── references/
│       ├── common-cases.md
│       ├── java-cases.md
│       ├── python-cases.md
│       └── php-cases.md
├── sql-injection-check/
│   ├── SKILL.md
│   └── references/
│       ├── common-cases.md
│       ├── java-cases.md
│       ├── python-cases.md
│       └── php-cases.md
├── xss-check/
│   ├── SKILL.md
│   └── references/
│       ├── common-cases.md
│       ├── java-cases.md
│       ├── python-cases.md
│       ├── php-cases.md
│       └── javascript-cases.md
├── unsafe-deserialization-check/
│   ├── SKILL.md
│   └── references/
│       ├── common-cases.md
│       ├── java-cases.md
│       ├── python-cases.md
│       └── php-cases.md
├── business-logic-abuse-check/
│   ├── SKILL.md
│   └── references/
│       ├── common-cases.md
│       ├── payment-cases.md
│       ├── authentication-cases.md
│       ├── rate-limit-cases.md
│       ├── workflow-cases.md
│       ├── promotion-cases.md
│       ├── resource-consumption-cases.md
│       └── third-party-integration-cases.md
├── command-execution-check/
│   ├── SKILL.md
│   └── references/
│       ├── common-cases.md
│       ├── java-cases.md
│       ├── python-cases.md
│       └── php-cases.md
├── path-traversal-check/
│   ├── SKILL.md
│   └── references/
│       ├── common-cases.md
│       ├── java-cases.md
│       ├── python-cases.md
│       └── php-cases.md
└── ssrf-check/
    ├── SKILL.md
    └── references/
        ├── common-cases.md
        ├── java-cases.md
        ├── python-cases.md
        └── php-cases.md
```

## 2. Skill Coverage Summary

| Skill | Main Goal | Core Test Content |
|---|---|---|
| Access Control Check | Check whether identity, role, object scope, tenant scope, and business-state restrictions are correctly enforced. | Authentication boundary, function-level authorization, object-level authorization, business-context authorization, client-side only auth, consistency across routes/methods/layers. |
| SQL Injection Check | Check whether untrusted input can alter query logic, query structure, or SQL execution behavior. | Source-to-query flow, unsafe query construction, parameterization and binding, ORDER BY / LIMIT / table / column control, ORM/raw query misuse, second-order SQLi. |
| XSS Check | Check whether user-controlled content reaches browser-executable rendering contexts unsafely. | Reflected/stored/DOM XSS, unsafe template output, raw HTML sinks, script/attribute/URL contexts, markdown/rich-text rendering, frontend DOM/framework sinks, sanitizer misuse. |
| Unsafe Deserialization Check | Check whether untrusted or weakly trusted data is restored into dangerous objects or types. | Untrusted input to deserializer, dangerous deserialization sinks, magic/lifecycle triggers, gadget behavior, integrity boundary failures, framework/library misuse, second-order restore paths. |
| Business Logic Abuse Check | Check whether core business rules can be abused through wrong state, order, frequency, value, or beneficiary binding. | State transitions, workflow sequencing, idempotency/replay, rate/quota abuse, payment/accounting integrity, actor-target-beneficiary mismatch, promotions, callbacks, async jobs. |
| Command Execution Check | Check whether untrusted input can influence commands, shells, interpreters, eval paths, or external tools. | Command construction, shell injection, argument/option injection, dangerous process APIs, eval/expression execution, external tool misuse, wrapper/helper sinks, second-order execution. |
| Path Traversal Check | Check whether untrusted input can escape intended file/resource scope. | Arbitrary read, write, delete, overwrite, include/load paths, unsafe path joins, normalize vs canonical path, base-dir containment, symlink issues, archive extraction / zip slip. |
| SSRF Check | Check whether untrusted input can influence outbound requests to unintended internal or privileged targets. | URL/host construction, outbound request sinks, internal network reachability, metadata access, redirect bypass, DNS/parser bypass, protocol misuse, indirect SSRF via preview/import/webhook/jobs. |

## 3 usage verification

### Access Control Check

/evidence/access-control-audit-report-v2.md

![image-20260420162019409](/Users/test/Library/Application Support/typora-user-images/image-20260420162019409.png)

---

## Need YouR Help

still developing，please give a star if it helps. Most of this project was completed with the help of AI, so  if it succeeds or fails, please file an issue for me.
