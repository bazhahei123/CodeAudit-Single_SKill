# PHP File Path Handling Source Cases

## Purpose

This file contains PHP-specific source point patterns and audit cases for path traversal and unsafe file path handling source discovery.

Use it when the target application is primarily implemented in PHP, especially in:
- Laravel
- Symfony
- raw PHP applications
- `file_get_contents`
- `fopen`
- `readfile`
- `unlink`
- `move_uploaded_file`
- include / require
- archive extraction
- PHP backends exposing download, upload, import/export, or local resource workflows

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream file operation in the target code.

---

# 1. PHP Source Discovery Points

Prioritize these source values and events:
- `$_GET`, `$_POST`, `$_REQUEST`, route parameters, request input, headers, and cookies
- uploaded original filenames, temporary file metadata, and extension values
- stored file paths from database, cache, session, queue, or job payloads
- archive entry names from zip/import helpers
- template names, include paths, theme selectors, and local resource names
- export destinations, upload destinations, cleanup targets, and delete targets
- framework storage disk/path values derived from request or stored metadata

Source questions:
- Which source supplies the filename, path, template name, or resource selector?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value decoded, normalized, joined, passed through `realpath`, or mapped before use?
- Which operation context should be audited next: read, write, delete, move, extract, include, or resource load?
- Which follow-up control should inspect containment, allowlists, wrapper behavior, or storage disk constraints?

---

# 2. PHP Source Patterns

## H-S1. Request-derived file or path value
Example idea:
- request parameter, route value, form field, or body value becomes `$file`, `$path`, `$template`, or `$name`.

Audit relevance:
Direct request-derived path values often feed reads, downloads, includes, exports, or cleanup paths.

Follow-up:
- trace the value into `file_get_contents`, `readfile`, `fopen`, `include`, storage helpers, or path joins.

## H-S2. Uploaded filename source
Example idea:
- uploaded original filename or extension influences save destination.

Audit relevance:
Upload metadata is client-controlled and can become write or second-order read source material.

Follow-up:
- verify server-generated filenames or strict filename allowlists.

## H-S3. Stored path or queue payload source
Example idea:
- database path, cached path, session value, or queue job payload is later used by file helpers.

Audit relevance:
Stored values may be second-order path sources when earlier attacker influence exists.

Follow-up:
- identify writer path and revalidation before file operations.

## H-S4. Include or template selector source
Example idea:
- request value selects PHP include, template path, theme path, locale file, or local resource.

Audit relevance:
Selectors can become local include/resource paths when not mapped through strict allowlists.

Follow-up:
- verify fixed template identifiers and no raw path include behavior.

## H-S5. Archive entry source
Example idea:
- zip entry or import manifest path influences extraction destination.

Audit relevance:
Archive entry names are path sources for extraction and overwrite behavior.

Follow-up:
- verify per-entry extraction-root containment and overwrite controls.

---

# 3. Case Templates

## Case H-S-PATH-1: Download/read path source

Source focus:
Identify request values, stored paths, or route parameters that reach read/download helpers.

Recommended follow-up:
Verify resolved containment and safe file read context.

## Case H-S-PATH-2: Upload destination source

Source focus:
Identify uploaded original filename, extension, storage disk, and destination path construction.

Recommended follow-up:
Verify server-generated names, allowlists, and overwrite behavior.

## Case H-S-PATH-3: Include/template source

Source focus:
Identify template/theme/resource values and how they map to include or render paths.

Recommended follow-up:
Verify strict resource-key mapping and no raw local include path control.

## Case H-S-PATH-4: Archive/import source

Source focus:
Identify archive entry names, import manifest paths, and extraction destination assembly.

Recommended follow-up:
Verify extraction-root containment for every entry.

---

# 4. PHP-Specific Audit Heuristics

## 4.1 Request and upload source heuristics
Pay attention to:
- `$_GET`
- `$_POST`
- `$request->input(...)`
- route params
- uploaded original filename
- content-disposition filename
- extension or MIME-derived path values

## 4.2 File API source heuristics
Pay attention to:
- `file_get_contents`
- `fopen`
- `readfile`
- `file_put_contents`
- `unlink`
- `rename`
- `copy`
- values passed into wrapper helpers

## 4.3 Include/resource source heuristics
Pay attention to:
- `include`
- `require`
- template path selection
- local resource loading
- theme, language, or view selectors

## 4.4 Upload, storage, and archive source heuristics
Pay attention to:
- `move_uploaded_file`
- framework storage path helpers
- export path logic
- cleanup and overwrite logic
- zip extraction helpers
- archive entry names or manifest paths

## 4.5 Layer inconsistency heuristics
Check whether path source handling is consistent across:
- preview vs download
- read vs delete/write
- upload destination vs cleanup path
- normal route vs admin/import path
- resource include vs file read helper

---

# 5. False-Positive Controls

Do not mark a PHP source as high-priority if:
- the value is selected from a strict allowlist of safe resource keys,
- the uploaded filename is replaced by a server-generated name before use,
- the value never reaches path construction or a file/resource operation,
- the stored path is trusted-only and cannot be attacker-influenced,
- the value is only used for display, logging, or labels.

Use `Suspected source` or `Not enough evidence` if:
- framework storage helper behavior is hidden,
- `realpath` or containment logic is in a shared helper not visible,
- archive entry handling is abstracted,
- the source is visible but operation context is unclear,
- include/template mapping may exist elsewhere.

---

# 6. What Good Evidence Looks Like

Good PHP source evidence includes:
- route/controller/script/worker/import entry point,
- source API such as `$_GET`, request input, upload filename, archive entry, cache read, or queue receive,
- propagation such as decode, concat, join, `realpath`, storage, or mapping,
- file API, storage helper, include/template helper, archive helper, or cleanup helper receiving the value,
- operation context when visible.

Good source evidence answers:
1. Which PHP entry point receives the path-like value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which path construction or file/resource operation should be audited next?
4. Is the source used for read, write, delete, move, extract, include, or resource load?

---

# 7. Quick PHP Source Checklist

- Are request values used as filenames, resource keys, templates, or path fragments?
- Are uploaded filenames or extensions used for save paths?
- Are stored paths read from database, cache, session, or queue payloads?
- Are include/template selectors mapped safely or used as raw paths?
- Are archive entry names or import manifest paths used for extraction?
- Are delete, move, copy, export, cleanup, and read paths sourced differently?
