# Python Unsafe Deserialization Source Cases

## Purpose

This file contains Python-specific source point patterns and audit cases for unsafe deserialization source discovery.

Use it when the target application is primarily implemented in Python, especially in:
- Django
- Flask
- FastAPI
- pickle-based workflows
- PyYAML loaders
- cache/session restore helpers
- queue consumers
- Python backends exposing import, background-job, or object-restore functionality

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream restore behavior in the target code.

---

# 1. Python Source Discovery Points

Prioritize these source values and events:
- request body bytes, `request.data`, `request.get_data()`, `request.body`, and uploaded files
- cookies, sessions, signed-looking blobs, and cache values
- queue message bodies, Celery task payloads, worker inputs, and background job arguments
- YAML text, YAML tags, pickle blobs, binary payloads, and custom decoder input
- base64-encoded, compressed, JSON-wrapped, or stored serialized data
- database/cache/session blobs later passed to `pickle`, `yaml.load`, or helper wrappers
- object reconstruction values that may influence `__reduce__`, `__setstate__`, factories, or post-restore logic

Source questions:
- Which source supplies bytes or text to pickle, YAML, or custom restore helpers?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the payload decoded, decompressed, or loaded from storage before restore?
- Are YAML tags, object constructors, or pickle reduction data attacker-influenced?
- Which queue, cache, session, or worker path should be audited next?

---

# 2. Python Source Patterns

## P-S1. Request-derived pickle or binary payload
Example idea:
- request bytes, upload content, or form field becomes input to `pickle.loads`, `pickle.load`, or a wrapper.

Audit relevance:
Direct request-derived pickle-like payloads are high-priority source points.

Follow-up:
- verify whether native object restoration occurs and whether safe formats can replace it.

## P-S2. YAML payload and tag source
Example idea:
- uploaded or request-provided YAML text includes tags, constructors, or object-like structures.

Audit relevance:
YAML safety depends on loader selection and whether object construction is enabled.

Follow-up:
- verify `safe_load` or strict schema parsing rather than unsafe loader modes.

## P-S3. Cookie/session/cache blob source
Example idea:
- cookie, session value, cache entry, or database field is decoded and restored.

Audit relevance:
Signed-looking or internal-looking blobs require verified integrity and safe restoration.

Follow-up:
- verify signing/MAC, safe serializer choice, and attacker influence on stored values.

## P-S4. Queue or worker message source
Example idea:
- worker consumes `message.body`, Celery payload, or background job argument and restores objects.

Audit relevance:
Internal queue transport is not automatically trusted if upstream producers are attacker-influenced.

Follow-up:
- verify producer trust, serializer settings, and safe task argument formats.

## P-S5. Reconstruction state source
Example idea:
- restored object state includes file paths, module names, callable names, URLs, or command-like values.

Audit relevance:
Reconstruction and post-restore behavior may be influenced by object state.

Follow-up:
- inspect `__reduce__`, `__setstate__`, object factories, and downstream consumers.

---

# 3. Case Templates

## Case P-S-DESER-1: Pickle source

Source focus:
Identify request, upload, cache, session, queue, or stored value passed toward `pickle.loads` or wrappers.

Recommended follow-up:
Verify trusted origin and whether pickle can be avoided for attacker-influenced data.

## Case P-S-DESER-2: YAML loader source

Source focus:
Identify YAML text origin, tag values, loader mode, and helper wrappers.

Recommended follow-up:
Verify safe loader use and strict schema constraints.

## Case P-S-DESER-3: Second-order cache or stored blob source

Source focus:
Identify writer path, cache/database blob, later reader, and restore helper.

Recommended follow-up:
Verify attacker influence and integrity controls across both paths.

## Case P-S-DESER-4: Queue message source

Source focus:
Identify queue producer, message body, task serializer, worker consumer, and restore helper.

Recommended follow-up:
Verify producer trust, serializer restrictions, and safe worker input formats.

---

# 4. Python-Specific Audit Heuristics

## 4.1 Pickle source heuristics
Pay attention to:
- `request.data`
- uploaded files
- `pickle.loads`
- `pickle.load`
- helper wrappers around pickle
- cache/session restore helpers
- hidden pickle use in utilities

## 4.2 YAML source heuristics
Pay attention to:
- YAML text origin
- YAML tag values
- loader class selection
- `yaml.load` wrappers
- import or config upload flows

## 4.3 Object reconstruction source heuristics
Pay attention to:
- `__reduce__`
- `__setstate__`
- object factories
- module/callable names
- file, URL, command, template, or callback-like properties

## 4.4 Queue / cache / session source heuristics
Pay attention to:
- message consumers
- Celery/task serializer settings
- cache reload helpers
- signed cookie/session serializers
- trust assumptions about "internal" data

## 4.5 Layer inconsistency heuristics
Check whether source handling is consistent across:
- request path vs worker path
- import path vs session/cache restore
- public API vs internal consumer
- safe loader path vs legacy helper path

---

# 5. False-Positive Controls

Do not mark a Python source as high-priority if:
- the value never reaches pickle, unsafe YAML, object restoration, or restore helpers,
- YAML is parsed with `safe_load` or strict primitive schema only,
- pickle is used only on trusted server-constructed data,
- cache/session data has verified integrity and safe serializer constraints,
- queue/task payloads use safe primitive formats and trusted producers.

Use `Suspected source` or `Not enough evidence` if:
- helper wrappers hide the actual loader,
- Celery/session serializer configuration is not visible,
- stored blob writer paths are missing,
- signing verification may exist elsewhere,
- reconstruction behavior is plausible but not visible.

---

# 6. What Good Evidence Looks Like

Good Python source evidence includes:
- route, view, worker, import handler, or cache/session helper entry point,
- source API such as `request.data`, upload read, cookie/session access, cache read, or queue receive,
- propagation such as base64 decode, decompress, storage, or task serialization,
- `pickle`, YAML loader, or restore helper receiving the value,
- YAML tags, object state, or reconstruction fields when visible.

Good source evidence answers:
1. Which Python entry point receives the serialized or restore-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which pickle, YAML, queue, cache, or session path should be audited next?
4. Is reconstruction-relevant object state influenced?

---

# 7. Quick Python Source Checklist

- Are request bodies, uploads, cookies, or sessions decoded and restored?
- Are base64/compressed values passed toward pickle or loader helpers?
- Are YAML tags or loader modes influenced by user-provided content?
- Are cache/database blobs later unpickled or loaded?
- Are queue/Celery payloads decoded into objects?
- Do restored values include file paths, URLs, commands, module names, or callables?
- Are worker and internal paths using the same source trust assumptions as public routes?
