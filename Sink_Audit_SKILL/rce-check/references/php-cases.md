# PHP Command Execution Cases

## Purpose

This file contains PHP-specific command execution patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in PHP, especially in:
- Laravel
- Symfony
- raw PHP applications
- `system`, `exec`, `shell_exec`, `passthru`
- `proc_open`, `popen`
- `eval`
- PHP backends exposing admin tools, automation, conversion, or external-tool workflows

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real execution boundary in the target code.

---

# 1. PHP Execution Control Points

When auditing PHP applications, prioritize these control points.

## 1.1 Direct process execution
Look for:
- `system`
- `exec`
- `shell_exec`
- `passthru`
- backticks
- `proc_open`
- `popen`
- wrapper helpers around shell or process launch

## 1.2 Shell and argument handling
Look for:
- concatenated shell strings
- dynamic command names
- user-controlled flags, file paths, or modes
- helper methods that wrap shell execution

## 1.3 Eval and dynamic execution
Look for:
- `eval`
- dynamic PHP code generation
- admin/debug evaluation helpers
- expression-style wrappers that may execute code

## 1.4 Tool-wrapper and job paths
Look for:
- image/PDF/archive/media conversion
- shell scripts
- cron/admin helpers
- queued jobs or replay tools that run CLIs

---

# 2. PHP Command Execution Anti-Patterns

### A1. `system` with concatenated user input
```php
system("ping " . $host);
```

Why risky:
User-controlled input directly influences command execution.

### A2. `shell_exec` on dynamic command string
```php
$output = shell_exec($cmd);
```

Why risky:
If `$cmd` is attacker-influenced, shell parsing may enable injection.

### A3. Dynamic command selection
```php
exec($toolName . " " . $filePath);
```

Why risky:
Attacker influence over command name changes what program runs.

### A4. Option injection
```php
exec("tar " . $userArg . " " . $archivePath);
```

Why risky:
Attacker input may be interpreted as a dangerous option or mode.

### A5. Unsafe `eval`
```php
eval($code);
```

Why risky:
Attacker-controlled code may execute directly.

### A6. Stored command template executed later
```php
$cmd = $task->command_template;
shell_exec($cmd);
```

Why risky:
This creates a second-order execution path if the stored template is attacker-influenced.

---

# 3. Case Templates

## Case H-CMD-1: Direct shell command injection

### Vulnerable pattern
```php
system("ping " . $host);
```

### Audit focus
Verify attacker influence on `$host` and whether shell interpretation is involved.

## Case H-CMD-2: Shell wrapper with dynamic command

### Vulnerable pattern
```php
$output = shell_exec($cmd);
```

### Audit focus
Trace how `$cmd` is built and whether fixed commands or allowlists exist.

## Case H-CMD-3: Unsafe PHP eval

### Vulnerable pattern
```php
eval($code);
```

### Audit focus
Verify whether `$code` is externally influenced and whether any safe subset is enforced.

## Case H-CMD-4: Helper-based tool execution

### Vulnerable pattern
```php
$runner->run($tool, $args);
```

### Audit focus
Trace the helper to the real sink and verify shell, argv, and allowlist behavior.

---

# 4. PHP-Specific Audit Heuristics

## 4.1 Process API heuristics
Pay attention to:
- `system`
- `exec`
- `shell_exec`
- `passthru`
- backticks
- `proc_open`
- `popen`
- wrapper helpers

## 4.2 Shell heuristics
Pay attention to:
- concatenated shell strings
- dynamic command names
- user-controlled option strings
- shell script invocation
- helper methods that claim to "sanitize" commands

## 4.3 Eval heuristics
Pay attention to:
- `eval`
- generated PHP code
- dynamic include/execute helpers
- admin/debug evaluation features

## 4.4 External tool heuristics
Pay attention to:
- image/PDF/archive/media conversion tools
- shell scripts
- cron/admin operational helpers
- batch/replay tools launching the same CLIs

## 4.5 Layer inconsistency heuristics
Check whether execution safety is consistent across:
- request path vs queue/replay path
- direct sink vs wrapper helper
- normal route vs admin path
- fixed-command path vs dynamic-template path
