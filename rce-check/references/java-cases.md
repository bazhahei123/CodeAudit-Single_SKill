# Java Command Execution Cases

## Purpose

This file contains Java-specific command execution patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- `Runtime.exec`
- `ProcessBuilder`
- `ScriptEngineManager`
- Groovy, SpEL, OGNL, MVEL, JEXL, or similar expression engines
- Java backends exposing admin tools, diagnostics, conversion, or external-tool workflows

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real execution boundary in the target code.

---

# 1. Java Execution Control Points

When auditing Java applications, prioritize these control points.

## 1.1 Direct process execution
Look for:
- `Runtime.getRuntime().exec(...)`
- `ProcessBuilder`
- wrapper services around process invocation
- shell wrappers
- OS utility helpers

## 1.2 Shell and interpreter invocation
Look for:
- `sh -c`
- `bash -c`
- `cmd.exe /c`
- `powershell -Command`
- script files launched from Java
- helper methods that hide shell usage

## 1.3 Expression and script execution
Look for:
- `ScriptEngineManager`
- Groovy execution helpers
- SpEL / OGNL / MVEL / JEXL evaluation
- dynamic rule or expression engines
- code generation or expression parsing from request data

## 1.4 Tool-wrapper and job paths
Look for:
- conversion or processing helpers
- network utility wrappers
- OCR / PDF / office / archive / image tools
- queued tasks that eventually launch external programs

---

# 2. Java Command Execution Anti-Patterns

### A1. Concatenated `Runtime.exec`
```java
Runtime.getRuntime().exec("ping " + host);
```

Why risky:
User-controlled input directly influences command execution.

### A2. Shell wrapper via `sh -c`
```java
new ProcessBuilder("sh", "-c", cmd).start();
```

Why risky:
The shell performs extra parsing and makes injection easier.

### A3. Dynamic command name
```java
new ProcessBuilder(toolName, filePath).start();
```

Why risky:
Attacker influence over `toolName` may control what program runs.

### A4. Option injection through `ProcessBuilder`
```java
new ProcessBuilder("tar", userArg, archivePath).start();
```

Why risky:
Even argv-style execution may be dangerous if `userArg` is an attacker-controlled option.

### A5. Script or expression evaluation
```java
Object result = engine.eval(expression);
```

Why risky:
Attacker-controlled expressions may lead to code execution or dangerous method access.

### A6. Stored command template later executed
```java
String cmd = task.getCommandTemplate();
new ProcessBuilder("sh", "-c", cmd).start();
```

Why risky:
This creates a second-order execution path if the template is attacker-influenced.

---

# 3. Case Templates

## Case J-CMD-1: Direct command injection

### Vulnerable pattern
```java
Runtime.getRuntime().exec("ping " + host);
```

### Audit focus
Verify whether `host` is attacker-controlled and whether command construction is string-based.

## Case J-CMD-2: Shell wrapper execution

### Vulnerable pattern
```java
new ProcessBuilder("sh", "-c", cmd).start();
```

### Audit focus
Treat attacker influence on `cmd` as high risk due to shell interpretation.

## Case J-CMD-3: Dangerous expression evaluation

### Vulnerable pattern
```java
Object result = engine.eval(expression);
```

### Audit focus
Verify whether `expression` can be influenced externally and whether the engine can access dangerous features.

## Case J-CMD-4: External tool wrapper misuse

### Vulnerable pattern
```java
processService.run(toolName, userArgs);
```

### Audit focus
Trace wrapper internals and verify fixed command / allowlist constraints.

---

# 4. Java-Specific Audit Heuristics

## 4.1 Process API heuristics
Pay attention to:
- `Runtime.exec`
- `ProcessBuilder`
- helper wrappers returning `Process`
- builder methods assembling commands dynamically

## 4.2 Shell heuristics
Pay attention to:
- `sh -c`
- `bash -c`
- `cmd /c`
- `powershell -Command`
- string-built shell payloads

## 4.3 Expression engine heuristics
Pay attention to:
- `ScriptEngineManager`
- Groovy execution
- SpEL / OGNL / MVEL / JEXL
- request-driven rule engines
- dynamic admin/debug evaluation features

## 4.4 External tool heuristics
Pay attention to:
- file conversion tools
- network tools
- archive helpers
- shell scripts
- wrappers that call CLIs indirectly

## 4.5 Layer inconsistency heuristics
Check whether execution safety is consistent across:
- request path vs queue/job path
- UI path vs admin/ops path
- direct sink vs wrapper helper
- fixed-command path vs configurable template path
