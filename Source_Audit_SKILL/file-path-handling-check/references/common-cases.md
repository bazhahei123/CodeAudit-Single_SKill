# File Path Handling Source Common Cases

## Purpose

This file defines shared path traversal and unsafe file path handling source concepts, source classification logic, propagation patterns, false-positive controls, and evidence standards that apply across languages and frameworks.

Use this file as the base reference for file path source review before loading any language-specific reference.

This file explains:
- what a file path source point is,
- how to distinguish source, propagation, path construction, sink, and control from a source-review perspective,
- how to classify trust boundaries,
- how to reason about direct and second-order path sources,
- when to record `Confirmed source`, `Suspected source`, `Not enough evidence`, or `Probably irrelevant`.

This reference is guidance, not proof. Do not report a vulnerability only because code contains a source described here. Always verify the real source origin, real path handling, and real downstream file/resource operation in the target code.

---

# 1. Core Concepts

## 1.1 File path source point

A file path source point is where data that may later influence a filesystem path, resource path, extraction path, upload/export destination, include path, or local resource selector enters or becomes available to the backend code path.

Examples:
- `file` query parameter
- uploaded filename
- archive entry name
- export filename
- template name
- stored path field
- queue job path
- cleanup target
- resource key
- user-selected report path

A source point is an audit starting point, not proof of a vulnerability.

## 1.2 Source, propagation, path construction, sink, and control

### Source
The origin of data that may be joined, normalized, mapped, resolved, or passed toward a file/resource operation.

Examples:
- `filename` from request body
- uploaded original filename
- `entry.getName()` from an archive
- `path` from a database row
- `template` from route parameter
- `message.path` from queue payload

### Propagation
The way a source is copied, decoded, normalized, joined, stored, renamed, extension-appended, mapped, or transformed before reaching a path construction point or sink.

Examples:
- URL decode before path join
- extension appended to a filename
- base directory joined with user path
- stored path later read by worker
- resource key mapped to template path
- path cleaned by helper

### Path construction
The point where the application turns a value into a filesystem or local resource path.

Examples:
- string concatenation
- `join` or `resolve`
- archive destination assembly
- template or resource path selection
- temporary file path generation using user input

### Sink
The file or resource operation using the path.

Examples:
- read
- write
- delete
- overwrite
- move / rename
- copy
- extract
- include / require
- template load
- local resource load

### Control
The logic intended to keep the source within safe scope.

Examples:
- strict resource-key allowlist
- server-generated filenames
- canonical containment check
- resolved base-directory comparison
- archive entry validation
- symlink-safe handling

Source discovery should document enough downstream use to explain why the source matters.

---

# 2. Trust Boundary Classification

## 2.1 Client-controlled
The value comes from request path, query string, request body, form data, headers, cookies, GraphQL variables, RPC arguments, uploaded filenames, uploaded file metadata, or user-supplied import rows.

## 2.2 External-system-controlled
The value comes from a queue message, webhook, partner payload, provider event, internal service response, file-processing job, or integration sync. It is only trusted after origin, integrity, and scope are verified.

## 2.3 Stored attacker-influenced
The value is read from database, cache, object storage, file metadata, saved reports, saved templates, import records, admin/replay storage, or job storage, and earlier attacker influence is possible.

## 2.4 Server-trusted
The value is constructed by trusted server-side code, generated as a safe filename, selected from a strict allowlist, or derived from trusted configuration with no attacker-controlled content.

## 2.5 Mixed
The value combines untrusted input with trusted data, or an untrusted value is mapped, allowlisted, or constrained before use.

## 2.6 Unclear
The origin cannot be determined from visible code.

---

# 3. Shared Source Categories

## 3.1 Direct path input sources
Values that directly look like filenames, paths, resource names, or path fragments.

Examples:
- `file`
- `filename`
- `path`
- `dir`
- `template`
- `resource`
- `download`

## 3.2 Uploaded filename and metadata sources
Values associated with uploaded files.

Examples:
- original filename
- content disposition filename
- MIME-derived extension
- archive name
- image/document metadata path

## 3.3 Archive and import entry sources
Values inside container formats or import bundles.

Examples:
- zip entry names
- tar member names
- nested archive paths
- import manifest paths
- package file names

## 3.4 Stored and second-order path sources
Values written earlier and used later as paths.

Examples:
- stored report path
- saved export filename
- cache file path
- database file path column
- queued cleanup target
- stored template name

## 3.5 Resource selector and include sources
Values that may not look like raw paths but select local resources.

Examples:
- theme name
- template key
- language file
- local resource name
- report type mapped to a path

---

# 4. Direct vs Second-Order Path Sources

## 4.1 Direct source path
The source reaches path construction or a file operation in the same request or immediate processing path.

Examples:
- download `file` parameter joined with base directory
- uploaded filename used as save destination
- request `template` used in local resource loader

## 4.2 Second-order source path
The source is stored first, then later used as a path in another code path.

Examples:
- uploaded original filename stored and later used for download
- export path saved to database and later cleaned up by worker
- archive entry name stored and later extracted
- import manifest path stored and later processed

Second-order source points should include both:
- where the value is written or influenced,
- where the value is later used as a path, when visible.

---

# 5. Shared Source Patterns

## S1. Request filename or path parameter
Example idea:
- route, query, form, or body field named `file`, `path`, `name`, `template`, or `resource`.

Audit relevance:
These sources often influence download, preview, include, export, or cleanup paths.

## S2. Uploaded original filename
Example idea:
- upload handler uses the client-provided filename or extension.

Audit relevance:
Upload metadata is client-controlled and can become write, overwrite, or later read source material.

## S3. Archive entry name
Example idea:
- zip/tar entry name is used to assemble extraction destination.

Audit relevance:
Every archive entry name behaves like attacker-controlled path input unless proven otherwise.

## S4. Stored path later reused
Example idea:
- database/cache/job value is read as file path later.

Audit relevance:
Storage does not make a path trusted when attacker influence exists upstream.

## S5. Resource selector mapped to path
Example idea:
- template, theme, locale, report type, or resource key maps to a local path.

Audit relevance:
Selector sources require follow-up to verify strict allowlists rather than raw path construction.

## S6. Cleanup or delete target
Example idea:
- job or API receives a path to delete, move, rename, or clean up.

Audit relevance:
Destructive operations need separate source tracking even when read paths are safe.

---

# 6. False-Positive Controls

Do not record a source point as high-priority if:
- the value is constant or constructed only by trusted server-side code,
- the value is selected from a strict allowlist of safe resource keys,
- the value is replaced with a server-generated filename before sensitive use,
- the downstream operation does not use the value as a path or resource selector,
- the source cannot be influenced by an attacker or weakly trusted producer.

Use `Suspected source` or `Not enough evidence` if:
- the source is visible but downstream file operation is hidden,
- the sink is visible but source origin is hidden,
- a helper wrapper may enforce containment but its implementation is unavailable,
- the value is stored and upstream influence is unclear,
- framework path resolution or storage abstraction behavior is not visible.

Do not over-claim based only on:
- parameter naming,
- file API presence,
- path joins,
- normalization calls,
- `../` test strings,
- an archive feature without visible untrusted entries.

---

# 7. Source Classification

## Confirmed source
Use `Confirmed source` when there is clear evidence that:
- the source origin is visible,
- it reaches path construction or a file/resource-operation-relevant path,
- and the trust boundary can be classified.

## Suspected source
Use `Suspected source` when:
- a value appears path-relevant,
- the origin or downstream use is partially visible,
- but hidden helper, framework, canonicalization, allowlist, storage, or wrapper behavior may change the classification.

## Not enough evidence
Use `Not enough evidence` when:
- the source origin cannot be determined,
- the downstream path use cannot be determined,
- or critical framework/storage behavior is not visible.

## Probably irrelevant
Use `Probably irrelevant` when:
- the value is visible,
- downstream use is visible,
- and it does not influence path construction, file/resource operations, archive extraction, include/load behavior, or stored path state.

---

# 8. What Good Evidence Looks Like

Strong source findings usually include:
- the exact entry point
- the source value name
- the source origin and trust boundary
- propagation steps such as decode, join, normalize, storage, or mapping
- the first downstream file-path-relevant use
- operation context if visible: read, write, delete, overwrite, move, extract, include, or resource load
- the follow-up sink/control check needed

Good source points usually answer:
1. Where does the path-like value enter?
2. Who controls or can influence it?
3. Is it direct, uploaded metadata, archive entry, stored, external, or resource selector?
4. What file path handling behavior should be checked next?
5. What code proves the connection?

---

# 9. Shared Follow-up Guidance

After source discovery, the path handling audit should verify:
- whether the source reaches a real file or resource sink,
- whether the path is constrained to a fixed base directory,
- whether canonical or resolved containment checks happen before use,
- whether symlink and check/use behavior are handled safely,
- whether archive entry names are validated per entry,
- whether strict allowlists or server-generated filenames replace raw input,
- whether read, write, delete, extract, include, and cleanup paths use equivalent controls.

Avoid weak conclusions such as:
- source exists, therefore vulnerability exists,
- normalization exists, therefore containment is safe,
- base directory is joined, therefore the path is safe,
- stored path exists, therefore it is trusted,
- one safe read path proves all write/delete paths are safe.

---

# 10. Shared Quick Checklist

Use this as a reminder, not as a substitute for reasoning.

- Is there a client-controlled or weakly trusted path-like source?
- Is the source a filename, full path, resource key, archive entry, or stored path?
- Is the source decoded, normalized, joined, mapped, or stored before use?
- Does it influence read, write, delete, overwrite, move, extract, include, or resource load behavior?
- Are uploaded filenames or archive entries treated as untrusted source values?
- Can stored values become second-order path sources?
- What sink and containment control should review this source next?
