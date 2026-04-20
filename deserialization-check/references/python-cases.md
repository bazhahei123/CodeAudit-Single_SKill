# Python Unsafe Deserialization Cases

## Purpose

This file contains Python-specific unsafe deserialization patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Python, especially in:
- Django
- Flask
- FastAPI
- pickle-based workflows
- PyYAML loaders
- cache/session restore helpers
- queue consumers
- Python backends exposing import, background-job, or object-restore functionality

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow, trust boundary, and dangerous post-deserialization behavior in the target code.

---

# 1. Python Deserialization Control Points

When auditing Python applications, prioritize these control points.

## 1.1 Entry and restore points
Look for:
- `pickle.loads`
- `pickle.load`
- `yaml.load`
- unsafe YAML loaders
- custom decoder helpers
- cache/session restore code
- queue consumers
- import handlers

## 1.2 Object behavior controls
Look for:
- `__reduce__`
- `__setstate__`
- custom reconstruction hooks
- object factories
- downstream code trusting restored object state

## 1.3 Framework and library controls
Look for:
- signed session or cookie blobs
- Celery / queue payload restoration
- cache deserialization
- YAML loader selection
- wrapper helpers that hide underlying restore logic

## 1.4 Trust-boundary controls
Look for:
- unsigned or weakly trusted cookies
- import payload trust assumptions
- internal queue or cache trust assumptions
- stored blob replay or later restore paths

---

# 2. Python Unsafe Deserialization Anti-Patterns

### A1. `pickle.loads` on request-derived data
```python
obj = pickle.loads(request.data)
```

Why risky:
Untrusted input reaches native Python object restoration directly.

### A2. Unsafe YAML loader
```python
data = yaml.load(payload, Loader=yaml.Loader)
```

Why risky:
Unsafe loaders may construct arbitrary Python objects.

### A3. Stored blob later unpickled
```python
blob = cache.get(key)
obj = pickle.loads(blob)
```

Why risky:
The attacker may influence stored data indirectly, creating a second-order path.

### A4. Implicit trust in queue message restore
```python
obj = pickle.loads(message.body)
```

Why risky:
Internal message transport does not automatically imply trusted origin.

### A5. Signed-looking but weakly trusted restore path
```python
payload = request.cookies.get("session_data")
obj = deserialize_session(payload)
```

Why risky:
The code may assume integrity without verifying it strongly.

---

# 3. Case Templates

## Case P-DESER-1: Direct pickle restore

### Vulnerable pattern
```python
obj = pickle.loads(request.data)
```

### Audit focus
Verify whether the input is attacker-controlled and whether dangerous object reconstruction is possible.

## Case P-DESER-2: Unsafe YAML load

### Vulnerable pattern
```python
data = yaml.load(payload, Loader=yaml.Loader)
```

### Audit focus
Verify whether the loader permits unsafe object construction and whether `payload` is attacker-controlled.

## Case P-DESER-3: Second-order cache or stored blob restore

### Vulnerable pattern
```python
blob = cache.get(key)
obj = pickle.loads(blob)
```

### Audit focus
Verify who can influence `blob` and whether integrity or trusted-origin controls exist.

## Case P-DESER-4: Queue message restore

### Vulnerable pattern
```python
obj = pickle.loads(message.body)
```

### Audit focus
Verify whether upstream producers can be attacker-influenced and whether safer formats or strict validation exist.

---

# 4. Python-Specific Audit Heuristics

## 4.1 Pickle heuristics
Pay attention to:
- `pickle.loads`
- `pickle.load`
- helper wrappers around pickle
- cache/session restore helpers
- hidden pickle usage in utilities

## 4.2 YAML heuristics
Pay attention to:
- `yaml.load`
- loader class selection
- safe vs unsafe loader modes
- helper wrappers that hide actual loader choice

## 4.3 Object behavior heuristics
Pay attention to:
- `__reduce__`
- `__setstate__`
- custom reconstruction side effects
- classes that can trigger external behavior after restore

## 4.4 Queue / cache heuristics
Pay attention to:
- message consumers
- Celery or background tasks
- cache reload helpers
- trust assumptions about "internal" data

## 4.5 Layer inconsistency heuristics
Check whether deserialization safety is consistent across:
- request path vs worker path
- import path vs session/cache restore
- public API vs internal consumer
- safe loader path vs legacy helper path
