# Java File Path Handling Source Cases

## Purpose

This file contains Java-specific source point patterns and audit cases for path traversal and unsafe file path handling source discovery.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- `java.io.File`
- `java.nio.file.Path` / `Paths`
- `Files.*`
- `ZipInputStream`
- resource or template loaders
- Java backends exposing file download, upload, import/export, or local resource workflows

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream file operation in the target code.

---

# 1. Java Source Discovery Points

Prioritize these source values and events:
- `@PathVariable`, `@RequestParam`, `@RequestBody`, `@RequestHeader`, and cookie values
- multipart uploaded original filenames and metadata
- `String`, `Path`, `File`, and resource name variables derived from request data
- archive entry names from `ZipEntry`, tar members, or import manifests
- stored file paths from database, cache, session, queue, or job payloads
- template names, report names, resource keys, and classpath/file resource selectors
- export destination, upload destination, cleanup target, and delete target values

Source questions:
- Which source supplies the filename, relative path, resource name, or archive entry?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value decoded, normalized, joined, resolved, or mapped before use?
- Which operation context should be audited next: read, write, delete, move, extract, or resource load?
- Which follow-up control should inspect canonical containment, allowlist, or symlink behavior?

---

# 2. Java Source Patterns

## J-S1. Request-derived filename or path
Example idea:
- request parameter, route variable, or DTO field becomes `fileName`, `path`, `template`, or `resource`.

Audit relevance:
Direct request-derived path values often feed download, preview, export, or local resource paths.

Follow-up:
- trace the value into `Paths.get`, `resolve`, string concatenation, `File`, or resource loaders.

## J-S2. Multipart uploaded filename source
Example idea:
- `MultipartFile.getOriginalFilename()` influences save path or later stored path.

Audit relevance:
Original filenames are client-controlled metadata and can become write or second-order read sources.

Follow-up:
- verify server-generated filenames or strict filename allowlists.

## J-S3. Archive entry name source
Example idea:
- `ZipEntry.getName()` or import manifest path is joined with extraction destination.

Audit relevance:
Archive entry names are path sources for extraction and overwrite behavior.

Follow-up:
- verify per-entry resolved extraction-root containment.

## J-S4. Stored path or queue payload source
Example idea:
- database path, cache value, or queue job argument is passed into file helpers.

Audit relevance:
Stored values may be second-order path sources when earlier attacker influence exists.

Follow-up:
- identify writer path and revalidation before file operations.

## J-S5. Resource or template selector source
Example idea:
- request value selects `Resource`, template, report, classpath item, or local file resource.

Audit relevance:
Selectors can become local resource paths if not mapped through strict allowlists.

Follow-up:
- verify safe resource-key mapping and no raw `file:` path construction.

---

# 3. Case Templates

## Case J-S-PATH-1: Download path source

Source focus:
Identify route/query/body values that become `Path`, `File`, or `Resource` inputs for download or preview.

Recommended follow-up:
Verify resolved base-directory containment and safe read operation context.

## Case J-S-PATH-2: Upload destination source

Source focus:
Identify uploaded original filename, extension, storage key, and destination path construction.

Recommended follow-up:
Verify server-generated names, allowlists, and overwrite behavior.

## Case J-S-PATH-3: Archive entry source

Source focus:
Identify archive entry names and destination path assembly.

Recommended follow-up:
Verify extraction-root containment for every entry.

## Case J-S-PATH-4: Cleanup/delete source

Source focus:
Identify delete, move, rename, or cleanup target source from request, storage, or queue payload.

Recommended follow-up:
Verify destructive operations use the same strong containment rules as reads.

---

# 4. Java-Specific Audit Heuristics

## 4.1 Request and DTO source heuristics
Pay attention to:
- `@PathVariable`
- `@RequestParam`
- `@RequestBody` DTO fields
- `@RequestHeader`
- multipart file metadata
- route names such as download, preview, export, import, delete, cleanup

## 4.2 `Path` / `File` source heuristics
Pay attention to:
- `Paths.get(...)`
- `resolve(...)`
- `normalize()`
- `toRealPath()`
- `new File(...)`
- string path concatenation
- source values passed before or after normalization

## 4.3 `Files.*` source heuristics
Pay attention to:
- `readAllBytes`
- `readString`
- `copy`
- `move`
- `delete`
- `exists`
- temp-file and cleanup logic

## 4.4 Spring/resource source heuristics
Pay attention to:
- `Resource` handlers
- `resourceLoader.getResource(...)`
- `FileSystemResource`
- classpath vs file resource selection
- template/resource lookup by request-derived names

## 4.5 Archive and worker source heuristics
Pay attention to:
- `ZipInputStream`
- `ZipEntry.getName()`
- import manifest paths
- queue jobs carrying file paths
- background workers that reprocess stored paths

---

# 5. False-Positive Controls

Do not mark a Java source as high-priority if:
- the value is selected from a strict allowlist of safe resource keys,
- the uploaded filename is replaced by a server-generated name before use,
- the value never reaches path construction or a file/resource operation,
- the stored path is trusted-only and cannot be attacker-influenced,
- the value is only used for display, logging, or labels.

Use `Suspected source` or `Not enough evidence` if:
- wrapper helper behavior is hidden,
- storage abstraction path resolution is not visible,
- archive entry handling is abstracted,
- the source is visible but operation context is unclear,
- canonical containment may exist elsewhere.

---

# 6. What Good Evidence Looks Like

Good Java source evidence includes:
- route/controller/worker/import entry point,
- source annotation or API such as `@RequestParam`, `MultipartFile`, `ZipEntry`, cache read, or queue receive,
- propagation such as decode, join, normalize, resolve, or storage,
- `Path`, `File`, `Resource`, `Files.*`, archive, or template helper receiving the value,
- operation context when visible.

Good source evidence answers:
1. Which Java entry point receives the path-like value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which path construction or file/resource operation should be audited next?
4. Is the source used for read, write, delete, move, extract, or resource load?

---

# 7. Quick Java Source Checklist

- Are request values used as filenames, resource keys, or path fragments?
- Is `MultipartFile.getOriginalFilename()` used for save or later paths?
- Are archive entry names joined with extraction destinations?
- Are stored paths read from database, cache, session, or queue payloads?
- Are template/resource selectors mapped safely or used as raw paths?
- Are delete, move, copy, export, cleanup, and read paths sourced differently?
