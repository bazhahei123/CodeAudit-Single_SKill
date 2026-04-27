# Python Command Execution Cases

## Purpose

This file contains Python-specific command execution patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Python, especially in:
- Django
- Flask
- FastAPI
- `os.system`
- `os.popen`
- `subprocess`
- `eval` / `exec`
- Python backends exposing admin tools, conversion, automation, or external-tool workflows

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real execution boundary in the target code.

---

# 1. Python Execution Control Points

When auditing Python applications, prioritize these control points.

## 1.1 Direct process execution
Look for:
- `os.system`
- `os.popen`
- `subprocess.run`
- `subprocess.call`
- `subprocess.Popen`
- helper wrappers around process execution

## 1.2 Shell and argument handling
Look for:
- `shell=True`
- string command arguments
- dynamic command selection
- user-controlled flags or file paths
- helper functions that wrap subprocess

## 1.3 Eval and dynamic execution
Look for:
- `eval`
- `exec`
- dynamic imports
- expression execution helpers
- debug/admin evaluation features

## 1.4 Tool-wrapper and job paths
Look for:
- OCR or conversion helpers
- archive or media-processing tools
- queued jobs launching shell or CLI utilities
- retry/replay paths that relaunch tools

---

# 2. Python Command Execution Anti-Patterns

### A1. `os.system` with user input
```python
os.system("ping " + host)
```

Why risky:
User-controlled input directly influences command execution.

### A2. `subprocess` with `shell=True`
```python
subprocess.run(f"ping {host}", shell=True)
```

Why risky:
The shell performs parsing and increases injection risk.

### A3. Dynamic command selection
```python
subprocess.run([tool_name, file_path])
```

Why risky:
Attacker influence over `tool_name` can change which program runs.

### A4. Option injection in argv mode
```python
subprocess.run(["tar", user_arg, archive_path])
```

Why risky:
Even without shell parsing, attacker-controlled options can alter tool behavior.

### A5. Unsafe `eval` / `exec`
```python
result = eval(expression)
```

Why risky:
Attacker-controlled expressions may lead to code execution directly.

### A6. Stored task or template later executed
```python
cmd = task.command
subprocess.run(cmd, shell=True)
```

Why risky:
This creates a second-order execution path if `task.command` is attacker-influenced.

---

# 3. Case Templates

## Case P-CMD-1: Shell command injection

### Vulnerable pattern
```python
subprocess.run(f"ping {host}", shell=True)
```

### Audit focus
Verify attacker influence on `host` and shell context reachability.

## Case P-CMD-2: Dangerous argv execution

### Vulnerable pattern
```python
subprocess.run([tool_name, file_path])
```

### Audit focus
Verify whether `tool_name`, options, or argument ordering are allowlisted.

## Case P-CMD-3: Unsafe expression execution

### Vulnerable pattern
```python
result = eval(expression)
```

### Audit focus
Verify whether the expression is attacker-controlled and whether a safe subset exists.

## Case P-CMD-4: Wrapper-based tool execution

### Vulnerable pattern
```python
tool_runner.run(user_input)
```

### Audit focus
Trace the wrapper to the real sink and verify shell / argv / allowlist behavior.

---

# 4. Python-Specific Audit Heuristics

## 4.1 Process API heuristics
Pay attention to:
- `os.system`
- `os.popen`
- `subprocess.*`
- helper functions that build commands
- string vs list invocation style

## 4.2 Shell heuristics
Pay attention to:
- `shell=True`
- string-form subprocess commands
- shell script invocation
- user-controlled content passed through shell wrappers

## 4.3 Eval heuristics
Pay attention to:
- `eval`
- `exec`
- dynamically executed expressions
- admin/debug features evaluating Python expressions
- rule-engine style wrappers

## 4.4 External tool heuristics
Pay attention to:
- file conversion
- OCR / PDF / media tools
- archive and compression tools
- automation scripts
- queued workers launching CLIs

## 4.5 Layer inconsistency heuristics
Check whether execution safety is consistent across:
- request path vs background job
- direct subprocess call vs helper wrapper
- safe argv path vs shell fallback path
- normal route vs admin / replay / batch path
