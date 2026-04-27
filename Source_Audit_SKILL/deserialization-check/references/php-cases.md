# PHP Unsafe Deserialization Source Cases

## Purpose

This file contains PHP-specific source point patterns and audit cases for unsafe deserialization source discovery.

Use it when the target application is primarily implemented in PHP, especially in:
- Laravel
- Symfony
- raw PHP applications
- session or cookie object restore logic
- Phar-related code paths
- PHP backends exposing import, cache, metadata, or object-restore functionality

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream restore behavior in the target code.

---

# 1. PHP Source Discovery Points

Prioritize these source values and events:
- `$_GET`, `$_POST`, `$_REQUEST`, request body, route params, and form fields
- cookies, session values, signed/encoded tokens, and cache values
- uploaded files, archive metadata, image metadata, and user-controlled file paths
- base64-encoded, compressed, or JSON-wrapped serialized strings
- stored serialized blobs from database, cache, queue, or session stores
- payloads passed to `unserialize` or wrappers around it
- file path sources that may reach Phar-aware file operations in the active runtime
- properties restored into `__wakeup`, `__destruct`, `__toString`, or chained object behavior

Source questions:
- Which source supplies the serialized string or object blob?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value decoded or loaded from storage before restore?
- Can a user-controlled path reach Phar metadata processing in the deployed PHP/runtime configuration?
- Which magic-method or post-restore state should be audited next?

---

# 2. PHP Source Patterns

## H-S1. Request-derived serialized string
Example idea:
- request parameter, body field, or form value is passed toward `unserialize` or a wrapper.

Audit relevance:
Direct request-derived serialized strings are high-priority source points.

Follow-up:
- verify whether `unserialize` is used and whether `allowed_classes` or safe formats are enforced.

## H-S2. Cookie or session blob source
Example idea:
- cookie or session-like value is base64-decoded and restored.

Audit relevance:
Client-controlled cookies require strong integrity validation before any object restoration.

Follow-up:
- verify signing/MAC and object constraints before restore.

## H-S3. Stored cache or database blob
Example idea:
- cache, database, or queue returns serialized text that is later passed to restore helpers.

Audit relevance:
Stored attacker-influenced values create second-order source paths.

Follow-up:
- identify who can write the stored value and whether trust controls exist.

## H-S4. Phar path source
Example idea:
- user-controlled path reaches file operations that may process Phar metadata in the active PHP/runtime configuration.

Audit relevance:
The source may be a path string rather than an explicit serialized payload.

Follow-up:
- verify whether Phar wrappers are possible in the deployed runtime and whether uploads/imports can supply Phar metadata.

## H-S5. Magic-method property source
Example idea:
- restored object properties include file paths, URLs, command fragments, include paths, or template names.

Audit relevance:
Properties can influence `__wakeup`, `__destruct`, `__toString`, or gadget-like behavior.

Follow-up:
- inspect magic methods and chained object behavior.

---

# 3. Case Templates

## Case H-S-DESER-1: Direct unserialize source

Source focus:
Identify the request, cookie, file, or stored value passed toward `unserialize`.

Recommended follow-up:
Verify trusted origin, `allowed_classes`, and magic-method reachability.

## Case H-S-DESER-2: Encoded cookie/session source

Source focus:
Identify cookie/session value, base64 decode, JSON wrapper, signing check, and restore helper.

Recommended follow-up:
Verify strong integrity validation and safe non-object storage.

## Case H-S-DESER-3: Second-order stored blob source

Source focus:
Identify cache/database writer, stored blob, later reader, and restore helper.

Recommended follow-up:
Verify attacker influence and trust-boundary controls across both paths.

## Case H-S-DESER-4: Phar path source

Source focus:
Identify file path sources from request, upload metadata, import flows, or stored paths that reach file operations.

Recommended follow-up:
Verify wrapper restrictions, runtime behavior, and whether uploaded Phar metadata can be processed.

---

# 4. PHP-Specific Audit Heuristics

## 4.1 `unserialize` source heuristics
Pay attention to:
- request variables passed to helpers
- base64-decoded strings
- cache/session restore helpers
- queue payloads
- wrapper functions around `unserialize`

## 4.2 Magic-method source heuristics
Pay attention to:
- `__wakeup`
- `__destruct`
- `__toString`
- object properties carrying file paths, URLs, commands, templates, or include targets
- chained objects restored from nested serialized properties

## 4.3 Phar source heuristics
Pay attention to:
- user-controlled file paths
- upload/import workflows
- archive/image validation paths
- `file_exists`, `is_file`, `getimagesize`, and similar file operations when the runtime/configuration can process Phar metadata through them
- metadata paths stored and processed later

## 4.4 Cache / session source heuristics
Pay attention to:
- signed vs unsigned cookies
- session serializers
- cache value writers and readers
- stored serialized state
- admin replay/import tools

## 4.5 Layer inconsistency heuristics
Check whether source handling is consistent across:
- request path vs cache/session restore
- import path vs file/metadata processing
- public API vs admin tooling
- explicit `unserialize` path vs hidden helper path

---

# 5. False-Positive Controls

Do not mark a PHP source as high-priority if:
- the value never reaches `unserialize`, Phar-aware processing, or object restoration,
- the value is parsed only as primitives or arrays with no object restoration,
- the source is strongly integrity-verified and object classes are disallowed,
- the file path cannot use Phar wrappers in the deployed runtime or cannot reach attacker-controlled metadata,
- restored values are trusted-only and not attacker-influenced.

Use `Suspected source` or `Not enough evidence` if:
- helper behavior hides whether `unserialize` is called,
- cookie/session signing behavior is not visible,
- stored blob writer paths are missing,
- file wrapper configuration is unclear,
- magic-method trigger behavior is plausible but not visible.

---

# 6. What Good Evidence Looks Like

Good PHP source evidence includes:
- route, controller, script, import handler, or file operation entry point,
- source API such as `$_POST`, cookie access, request input, upload metadata, cache read, or queue body,
- propagation such as base64 decode or storage/reload,
- `unserialize` wrapper or file operation receiving the value,
- restored class or property source if visible.

Good source evidence answers:
1. Which PHP entry point receives the serialized, path, or restore-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which `unserialize`, Phar/runtime-specific metadata, session/cache, or magic-method path should be audited next?
4. Is trigger-relevant object state influenced?

---

# 7. Quick PHP Source Checklist

- Are request values, cookies, or sessions decoded and restored?
- Are base64 values passed toward `unserialize` wrappers?
- Are cache/database blobs later unserialized?
- Are user-controlled paths used in file operations that are Phar-aware in the deployed runtime?
- Are upload metadata or stored paths processed later?
- Do restored properties include file paths, URLs, commands, templates, or include targets?
- Are magic methods reachable from restored object state?
