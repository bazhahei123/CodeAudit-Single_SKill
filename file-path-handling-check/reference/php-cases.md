# PHP Path Traversal Cases

## Purpose

This file contains PHP-specific path traversal patterns, anti-patterns, and audit cases.

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

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real path containment behavior in the target code.

---

# 1. PHP Path Control Points

When auditing PHP applications, prioritize these control points.

## 1.1 File read and preview paths
Look for:
- `file_get_contents`
- `fopen`
- `readfile`
- download handlers
- preview handlers
- log/config readers
- local file/resource loaders

## 1.2 File write, move, and delete paths
Look for:
- `file_put_contents`
- `unlink`
- `rename`
- `copy`
- `move_uploaded_file`
- export/save path logic
- cleanup helpers

## 1.3 Path construction and validation
Look for:
- string path concatenation
- `realpath`
- custom path-cleaning helpers
- wrapper methods around storage/filesystem access
- include/resource path selection

## 1.4 Archive extraction and include/resource loading
Look for:
- zip extraction
- archive helpers
- `include`
- `require`
- template selection by path-like input
- local resource loading by user-controlled path

---

# 2. PHP Path Traversal Anti-Patterns

### A1. Direct read path from request
```php
readfile($baseDir . "/" . $_GET['file']);
```

Why risky:
User-controlled input may escape the intended base directory.

### A2. Unsafe delete path
```php
unlink($baseDir . "/" . $name);
```

Why risky:
Weak containment allows attacker-controlled deletion outside the intended scope.

### A3. Raw include path control
```php
include $templatePath;
```

Why risky:
User-controlled include/resource paths may expose unintended local files or cause unsafe local inclusion.

### A4. Weak `realpath` usage
```php
$path = realpath($baseDir . "/" . $userPath);
if (strpos($path, $baseDir) === 0) {
    echo file_get_contents($path);
}
```

Why risky:
Containment logic must be applied carefully and consistently across all operations.

### A5. Archive extraction without entry validation
```php
$zip->extractTo($destDir);
```

Why risky:
Archive entries may escape the intended extraction root.

---

# 3. Case Templates

## Case H-PATH-1: Arbitrary read via concatenated path

### Vulnerable pattern
```php
readfile($baseDir . "/" . $_GET['file']);
```

### Audit focus
Verify whether the input is attacker-controlled and whether resolved-path containment is enforced before use.

## Case H-PATH-2: Delete or overwrite path abuse

### Vulnerable pattern
```php
unlink($baseDir . "/" . $name);
```

### Audit focus
Verify canonical containment and whether destructive operations use the same safe path logic as reads.

## Case H-PATH-3: Local include/resource path control

### Vulnerable pattern
```php
include $templatePath;
```

### Audit focus
Verify whether raw path input can escape intended local resource scope.

## Case H-PATH-4: Unsafe archive extraction

### Vulnerable pattern
```php
$zip->extractTo($destDir);
```

### Audit focus
Verify extraction-root containment, entry validation, and overwrite behavior.

---

# 4. PHP-Specific Audit Heuristics

## 4.1 File API heuristics
Pay attention to:
- `file_get_contents`
- `fopen`
- `readfile`
- `file_put_contents`
- `unlink`
- `rename`
- `copy`
- wrapper helpers

## 4.2 Include/resource heuristics
Pay attention to:
- `include`
- `require`
- template path selection
- local resource loading
- raw PHP fallback templates

## 4.3 Upload and storage heuristics
Pay attention to:
- `move_uploaded_file`
- destination filename/path selection
- export path logic
- cleanup and overwrite logic

## 4.4 Archive heuristics
Pay attention to:
- zip extraction helpers
- archive entry trust
- destination assembly
- overwrite behavior

## 4.5 Layer inconsistency heuristics
Check whether path safety is consistent across:
- preview vs download
- read vs delete/write
- upload destination vs cleanup path
- normal route vs admin/import path
- resource include vs file read helper
