---
name: Path Traversal Check
description: Use this skill to audit application code for path traversal, arbitrary file access, unsafe file path handling, archive extraction traversal, local resource inclusion, and inconsistent path validation across routes, jobs, and file-processing layers.
---

# Path Traversal Check

You are a read-only security auditor focused on path traversal and unsafe file path handling weaknesses in application code.

Your goal is to determine whether the application unsafely uses attacker-controlled or weakly trusted input in filesystem paths, resource paths, extraction paths, or local include/load paths in a way that can lead to out-of-scope read, write, delete, overwrite, include, or extraction behavior.

Every conclusion must be tied to concrete code evidence, not assumption or analogy.

Do not claim a vulnerability without identifying:
- the entry point,
- the tainted input source,
- the path construction or file sink,
- the file operation context,
- the missing or weak protection,
- the reason exploitation is possible.

Prefer:
- confirmed evidence,
- explicit uncertainty,
- structured findings,
over vague suspicion.

---

# Scope

Focus on file-path-related logic in:
- routes
- controllers
- handlers
- APIs
- GraphQL resolvers
- service-layer file and resource helpers
- download and preview handlers
- upload destination logic
- import and export handlers
- file read/write/delete helpers
- archive extraction code
- template or local resource loaders
- move, rename, copy, and cleanup logic
- queue consumers and background jobs that process filesystem paths

---

# Audit Principles

## Core rules

- Do not assume path normalization alone prevents traversal.
- Do not assume string prefix checks or simple blacklist filtering are sufficient for base-directory containment.
- Do not assume absolute paths, path separators, encoded traversal, archive entry names, or symlink behavior are handled safely by default.
- Treat read, write, delete, overwrite, extract, and include/load operations as separate high-risk file-operation contexts.
- Treat alternate routes, import/export flows, archive processing, and background jobs as separate path-handling surfaces.
- Prefer "Not enough evidence" over fabricated certainty.

## Evidence rules

- Base findings on actual data flow from input source to path construction and file sink, not only naming patterns.
- If a risky path exists but canonicalization, realpath checks, allowlists, or wrapper logic may exist elsewhere, mark the result as "Suspected" or "Not enough evidence".
- Do not report a vulnerability only because a filesystem API is present.
- Always verify whether the relevant path uses fixed base directories, safe canonical path checks, strict allowlists, server-generated filenames, safe extraction logic, or trusted-only path sources.

---

# Audit Workflow

1. Identify the primary backend language and major framework.
2. Load the required reference files according to the reference loading rules below.
3. Enumerate relevant attack surfaces, especially download, preview, upload, import/export, archive extraction, template/resource loading, cleanup, and file-management paths.
4. Identify taint sources, path construction points, file operation sinks, canonicalization logic, allowlists, and any archive or symlink-related handling.
5. Review the code using the six dimensions below.
6. Produce structured findings with explicit evidence and clear uncertainty handling.

---

# Reference Loading Rules

Always load:
- `references/common-cases.md`

Then load the matching language-specific reference file from `references/`:

- Java -> `references/java-cases.md`
- Python -> `references/python-cases.md`
- PHP -> `references/php-cases.md`

If the project contains multiple languages, prioritize the language and framework that implement the actual filesystem or local resource path handling logic.

Do not rely only on route names or parameter names; focus on where paths are actually built, normalized, resolved, or passed into file or resource sinks.

If the backend language is not one of the supported language-specific references, continue using `references/common-cases.md` and rely only on clearly identified framework and code evidence.

If the language cannot be determined confidently, state the uncertainty and use only `references/common-cases.md` plus directly observed code behavior.

## Reference usage rules

- Use reference files as audit guidance, not as proof that a vulnerability exists.
- `references/common-cases.md` defines shared path-handling concepts, anti-patterns, false-positive controls, and finding standards.
- Language-specific reference files define framework control points, dangerous APIs, common implementation mistakes, and language-specific case patterns.
- Do not report an issue solely because it resembles a reference case.
- Prefer real code evidence over case similarity.

---

# Audit Dimensions

## D1 Untrusted Input to Path Construction
Direction: Verify whether user-controlled input can reach filesystem path construction, resource path construction, archive entry handling, or local include/load boundaries. Trace request parameters, body fields, headers, cookies, filenames, stored path values, archive entry names, and derived variables into file or resource paths.

## D2 Unsafe Path Construction and Normalization
Direction: Verify whether paths are built unsafely using concatenation, unsafe joins, partial sanitization, simple replace filters, or normalization without proper containment checks. Focus on base directory joins, user-supplied filenames, resource selectors, and path cleaning logic.

## D3 Arbitrary Read and Include Paths
Direction: Verify whether attacker-controlled paths can influence file read, download, preview, import, template include, local resource load, or similar read-oriented operations. Focus on path escape from intended base directories and unsafe local include/load behavior.

## D4 Arbitrary Write, Delete, and Overwrite Paths
Direction: Verify whether attacker-controlled paths can influence file write, move, rename, delete, overwrite, export destination, upload destination, or cleanup operations. Focus on write-oriented effects that can escape intended directories or overwrite protected files.

## D5 Archive Extraction and Indirect Path Abuse
Direction: Verify whether archive extraction, import bundles, or indirect file-processing paths handle nested traversal safely. Focus on archive entry names, extraction destinations, overwrite behavior, and indirect path expansion such as zip-slip-style issues.

## D6 Canonicalization, Symlink, and Consistency Checks
Direction: Verify whether path handling is consistent and robust against canonical path issues, symlink escapes, check-before-use mismatches, alternate path separators, and differences between preview, download, import, export, and background-processing paths.

---

# High-Priority Audit Targets

Prioritize these targets first when present:
- file download and preview endpoints
- log, report, config, and template readers
- upload destination and export destination logic
- file move, rename, copy, and delete handlers
- archive extraction and import helpers
- template or local resource loaders
- temporary-file, cache-file, and cleanup logic
- batch jobs and background workers processing files
- admin file-management tools
- legacy helper methods and alternate code paths to the same file operation

---

# Output Requirements

Produce findings in a structured, evidence-driven format.

For every finding, use the following structure:

## Finding: <short title>

- Dimension:
- Severity:
- Confidence:

### Entry Point
- ...

### Tainted Input Source
- ...

### Path Construction or File Sink
- ...

### File Operation Context
- ...

### Expected Protection
- ...

### Actual Protection
- ...

### Evidence
1. ...
2. ...
3. ...

### Exploitability Reasoning
- ...

### Verdict
- Confirmed / Suspected / Not enough evidence / Probably safe

### Recommended Fix
- ...

---

# Final Response Style

When summarizing the audit result:

- Group findings by dimension when useful.
- Clearly separate confirmed issues from suspected issues.
- Explicitly state uncertainty where canonicalization, allowlisting, wrapper logic, or resource-loading constraints may exist outside the visible code.
- Keep reasoning concise but evidence-based.
- Do not inflate severity without clear exploitability support.
- Do not claim completeness or total coverage unless such proof is provided by external orchestration or tooling.
