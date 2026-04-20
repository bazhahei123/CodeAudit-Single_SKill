# PHP Unsafe Deserialization Cases

## Purpose

This file contains PHP-specific unsafe deserialization patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in PHP, especially in:
- Laravel
- Symfony
- raw PHP applications
- session or cookie object restore logic
- Phar-related code paths
- PHP backends exposing import, cache, metadata, or object-restore functionality

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow, trust boundary, and dangerous post-deserialization behavior in the target code.

---

# 1. PHP Deserialization Control Points

When auditing PHP applications, prioritize these control points.

## 1.1 Entry and restore points
Look for:
- `unserialize`
- session restore helpers
- cookie object restore logic
- cache restore helpers
- metadata processing
- import handlers
- file functions that may trigger Phar metadata handling

## 1.2 Object behavior controls
Look for:
- `__wakeup`
- `__destruct`
- `__toString`
- custom magic-method side effects
- downstream code trusting restored object state

## 1.3 Framework and library controls
Look for:
- wrapper helpers around `unserialize`
- session serializers
- cache / queue object restore
- object restore hidden behind helper methods
- file operations that can trigger phar behavior

## 1.4 Trust-boundary controls
Look for:
- client-controlled serialized blobs
- unsigned or weakly trusted cookies
- stored object blob replay
- internal-only trust assumptions
- metadata or file path handling with hidden object restore effects

---

# 2. PHP Unsafe Deserialization Anti-Patterns

### A1. `unserialize` on request-derived data
```php
$obj = unserialize($_POST['data']);
```

Why risky:
Untrusted input reaches native PHP object restoration directly.

### A2. Base64-decoded object restore from client input
```php
$obj = unserialize(base64_decode($_COOKIE['session']));
```

Why risky:
Encoding does not provide trust or integrity.

### A3. Stored blob later unserialized
```php
$blob = $cache->get($key);
$obj = unserialize($blob);
```

Why risky:
The attacker may influence stored data indirectly, creating a second-order path.

### A4. Magic-method side effects after restore
```php
$obj = unserialize($payload);
```

Why risky:
Dangerous behavior may occur through `__wakeup`, `__destruct`, `__toString`, or chained object behavior.

### A5. Phar-triggered deserialization path
```php
file_exists($userControlledPath);
```

Why risky:
Some file operations can trigger dangerous metadata processing when phar paths are involved.

---

# 3. Case Templates

## Case H-DESER-1: Direct `unserialize`

### Vulnerable pattern
```php
$obj = unserialize($_POST['data']);
```

### Audit focus
Verify whether the input is attacker-controlled and whether dangerous object behavior is reachable.

## Case H-DESER-2: Cookie/session object restore

### Vulnerable pattern
```php
$obj = unserialize(base64_decode($_COOKIE['session']));
```

### Audit focus
Verify whether strong integrity protection exists and whether restored objects are constrained.

## Case H-DESER-3: Second-order cache or stored blob restore

### Vulnerable pattern
```php
$blob = $cache->get($key);
$obj = unserialize($blob);
```

### Audit focus
Verify who controls `blob` and whether the restore path assumes trust incorrectly.

## Case H-DESER-4: Phar-related path

### Vulnerable pattern
```php
file_exists($userControlledPath);
```

### Audit focus
Verify whether user-controlled paths can reference phar payloads and trigger dangerous metadata handling.

---

# 4. PHP-Specific Audit Heuristics

## 4.1 `unserialize` heuristics
Pay attention to:
- direct `unserialize`
- helper wrappers around `unserialize`
- base64-decoded object restore
- session / cookie restore code

## 4.2 Magic-method heuristics
Pay attention to:
- `__wakeup`
- `__destruct`
- `__toString`
- objects with filesystem, network, command, or include side effects
- chained object behavior after restore

## 4.3 Phar heuristics
Pay attention to:
- file operations on user-controlled paths
- metadata processing
- upload/import workflows
- image/archive validation paths that touch filesystem APIs

## 4.4 Cache / session heuristics
Pay attention to:
- stored serialized state
- cache reload helpers
- trust assumptions about server-side stored data
- signed vs unsigned restore paths

## 4.5 Layer inconsistency heuristics
Check whether deserialization safety is consistent across:
- request path vs cache/session restore
- import path vs file/metadata processing
- public API vs admin tooling
- explicit `unserialize` path vs hidden helper path
