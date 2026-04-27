# Java Path Traversal Cases

## Purpose

This file contains Java-specific path traversal patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- `java.io.File`
- `java.nio.file.Path` / `Paths`
- `Files.*`
- `ZipInputStream`
- resource or template loaders
- Java backends exposing file download, upload, import/export, or local resource workflows

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real path containment behavior in the target code.

---

# 1. Java Path Control Points

When auditing Java applications, prioritize these control points.

## 1.1 File read and preview paths
Look for:
- download handlers
- preview handlers
- `Files.readAllBytes`
- `FileInputStream`
- `Resource` loaders
- log or config readers

## 1.2 File write, move, and delete paths
Look for:
- upload destination logic
- export destination logic
- `Files.copy`
- `Files.move`
- `Files.delete`
- `FileOutputStream`
- cleanup helpers

## 1.3 Path construction and validation
Look for:
- `Paths.get(...)`
- `resolve(...)`
- `normalize()`
- `toRealPath()`
- string path concatenation
- wrapper helpers around file access

## 1.4 Archive extraction and resource loading
Look for:
- `ZipInputStream`
- archive entry name handling
- template/resource loading
- local include-like resource resolution

---

# 2. Java Path Traversal Anti-Patterns

### A1. Concatenated read path
```java
File f = new File(baseDir + "/" + fileName);
byte[] data = Files.readAllBytes(f.toPath());
```

Why risky:
User-controlled `fileName` may escape the intended base directory.

### A2. Normalize-only containment
```java
Path p = Paths.get(baseDir, userPath).normalize();
if (p.startsWith(baseDir)) {
    return Files.readString(p);
}
```

Why risky:
`normalize()` and simple prefix checks may not be sufficient depending on how containment is validated and used.

### A3. Unsafe delete path
```java
Files.delete(Paths.get(baseDir, userPath));
```

Why risky:
Weak containment allows attacker-controlled deletion outside the intended scope.

### A4. Zip extraction without entry validation
```java
Path out = Paths.get(destDir, entry.getName());
Files.copy(zipStream, out);
```

Why risky:
Archive entry names may traverse outside the extraction root.

### A5. Resource/template selection by raw path
```java
return resourceLoader.getResource("file:" + templatePath);
```

Why risky:
Local resource loading may expose unintended files if `templatePath` is attacker-controlled.

---

# 3. Case Templates

## Case J-PATH-1: Arbitrary read via joined path

### Vulnerable pattern
```java
Path p = Paths.get(baseDir, userPath);
return Files.readString(p);
```

### Audit focus
Verify whether `userPath` is attacker-controlled and whether real containment is enforced before use.

## Case J-PATH-2: Unsafe archive extraction

### Vulnerable pattern
```java
Path out = Paths.get(destDir, entry.getName());
Files.copy(zipStream, out);
```

### Audit focus
Verify entry-path containment and overwrite behavior during extraction.

## Case J-PATH-3: Delete or overwrite path abuse

### Vulnerable pattern
```java
Files.delete(Paths.get(baseDir, userPath));
```

### Audit focus
Verify canonical containment and whether delete/write operations share the same path validation rules as read operations.

## Case J-PATH-4: Template or local resource path control

### Vulnerable pattern
```java
resourceLoader.getResource("file:" + templatePath);
```

### Audit focus
Verify whether raw path input can escape intended local resource scope.

---

# 4. Java-Specific Audit Heuristics

## 4.1 `Path` / `Paths` heuristics
Pay attention to:
- `Paths.get(...)`
- `resolve(...)`
- `normalize()`
- `toRealPath()`
- differences between string checks and resolved-path checks

## 4.2 `Files.*` heuristics
Pay attention to:
- `readAllBytes`
- `readString`
- `copy`
- `move`
- `delete`
- `exists`
- temp-file and cleanup logic

## 4.3 Spring/resource heuristics
Pay attention to:
- download and `Resource` handlers
- file-serving endpoints
- template/resource lookup by name
- local file vs classpath resource confusion

## 4.4 Archive heuristics
Pay attention to:
- `ZipInputStream`
- archive entry path usage
- extraction-root enforcement
- overwrite behavior

## 4.5 Layer inconsistency heuristics
Check whether path safety is consistent across:
- preview vs download
- read vs delete/write
- upload destination vs cleanup path
- normal route vs admin/import path
