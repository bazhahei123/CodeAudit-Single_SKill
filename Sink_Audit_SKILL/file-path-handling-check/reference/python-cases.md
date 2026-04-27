# Python Path Traversal Cases

## Purpose

This file contains Python-specific path traversal patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Python, especially in:
- Django
- Flask
- FastAPI
- `open`
- `os.path`
- `pathlib`
- `zipfile` / `tarfile`
- file-serving helpers
- Python backends exposing download, upload, import/export, or local resource workflows

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real path containment behavior in the target code.

---

# 1. Python Path Control Points

When auditing Python applications, prioritize these control points.

## 1.1 File read and preview paths
Look for:
- `open(...)`
- `send_file`
- file preview helpers
- log/config readers
- import readers
- local template/resource loading

## 1.2 File write, move, and delete paths
Look for:
- upload destination logic
- export/save path logic
- `os.remove`
- `os.rename`
- `shutil.move`
- `open(..., 'w')`
- cleanup helpers

## 1.3 Path construction and validation
Look for:
- `os.path.join`
- `os.path.abspath`
- `os.path.realpath`
- `pathlib.Path`
- `resolve()`
- string path concatenation
- wrapper helpers

## 1.4 Archive extraction and resource loading
Look for:
- `zipfile`
- `tarfile`
- extraction destination assembly
- local file/resource loaders
- template selection by path-like input

---

# 2. Python Path Traversal Anti-Patterns

### A1. Unsafe joined read path
```python
path = os.path.join(base_dir, user_path)
with open(path, "rb") as f:
    return f.read()
```

Why risky:
User-controlled `user_path` may escape the intended base directory.

### A2. Weak absolute-path check
```python
path = os.path.abspath(os.path.join(base_dir, user_path))
if path.startswith(base_dir):
    return open(path).read()
```

Why risky:
Containment checks need careful real-path handling and consistent use.

### A3. Unsafe delete path
```python
os.remove(os.path.join(base_dir, user_path))
```

Why risky:
Weak containment allows attacker-controlled deletion outside the intended scope.

### A4. Tar/zip extraction without entry validation
```python
tar.extractall(dest_dir)
```

Why risky:
Archive entry names may escape the extraction root.

### A5. Local template/resource path control
```python
return send_file(user_path)
```

Why risky:
Raw path input may expose unintended files or resources.

---

# 3. Case Templates

## Case P-PATH-1: Arbitrary read via joined path

### Vulnerable pattern
```python
path = os.path.join(base_dir, user_path)
return open(path).read()
```

### Audit focus
Verify whether `user_path` is attacker-controlled and whether resolved-path containment is enforced before use.

## Case P-PATH-2: Unsafe archive extraction

### Vulnerable pattern
```python
tar.extractall(dest_dir)
```

### Audit focus
Verify entry validation, extraction-root containment, and overwrite behavior.

## Case P-PATH-3: Delete or overwrite path abuse

### Vulnerable pattern
```python
os.remove(os.path.join(base_dir, user_path))
```

### Audit focus
Verify whether delete/write operations use the same strong containment logic as read operations.

## Case P-PATH-4: File-serving helper misuse

### Vulnerable pattern
```python
return send_file(user_path)
```

### Audit focus
Verify whether raw user input can control file-serving paths.

---

# 4. Python-Specific Audit Heuristics

## 4.1 `os.path` heuristics
Pay attention to:
- `join`
- `abspath`
- `realpath`
- simple prefix checks
- wrapper helpers around path validation

## 4.2 `pathlib` heuristics
Pay attention to:
- `Path(...)`
- `resolve()`
- parent containment checks
- differences between resolved and unresolved paths

## 4.3 File-serving heuristics
Pay attention to:
- `send_file`
- Django/Flask/FastAPI download helpers
- preview paths
- import/export helpers
- local resource readers

## 4.4 Archive heuristics
Pay attention to:
- `zipfile`
- `tarfile`
- `extractall`
- manual entry extraction
- destination assembly and overwrite behavior

## 4.5 Layer inconsistency heuristics
Check whether path safety is consistent across:
- preview vs download
- read vs delete/write
- upload destination vs cleanup path
- normal route vs background/import path
