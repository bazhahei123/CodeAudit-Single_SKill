# Java Command Execution Source Cases

## Purpose

This file contains Java-specific source point patterns and audit cases for command execution and code execution source discovery.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- `Runtime.exec`
- `ProcessBuilder`
- `ScriptEngineManager`
- Groovy, SpEL, OGNL, MVEL, JEXL, or similar expression engines
- Java backends exposing admin tools, diagnostics, conversion, or external-tool workflows

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream execution behavior in the target code.

---

# 1. Java Source Discovery Points

Prioritize these source values and events:
- `@PathVariable`, `@RequestParam`, `@RequestBody`, headers, cookies, and multipart metadata
- command names, tool names, subcommands, script paths, and executable paths
- `List<String>` argv values, option values, flags, file paths, URLs, modes, and targets
- shell strings passed to `sh -c`, `bash -c`, `cmd.exe /c`, or PowerShell
- expression strings for `ScriptEngineManager`, Groovy, SpEL, OGNL, MVEL, JEXL, or rule engines
- environment maps, working directories, stdin values, config paths, and generated script files
- stored task definitions, queue payloads, job arguments, and admin replay data

Source questions:
- Which source supplies command name, tool name, shell string, argv value, option, or expression?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value formatted, quoted, joined, allowlisted, mapped, stored, or wrapped before use?
- Which execution context should be audited next: shell, argv, eval/expression, interpreter, external tool, or wrapper?

---

# 2. Java Source Patterns

## J-S1. Request-derived command argument
Example idea:
- request value such as host, path, URL, report ID, mode, or target becomes an argument to a process wrapper.

Audit relevance:
The value may affect command behavior, tool target, option parsing, or file access.

Follow-up:
- trace into `Runtime.exec`, `ProcessBuilder`, wrapper services, or external tool helpers.

## J-S2. Dynamic command or tool selector
Example idea:
- request or stored field selects `toolName`, script, subcommand, interpreter, or action.

Audit relevance:
Command/tool selection sources are high priority even when arguments are arrays.

Follow-up:
- verify fixed commands or strict allowlists.

## J-S3. Shell string or command template source
Example idea:
- formatted string, template, or stored command text is passed to `sh -c`, `cmd /c`, or shell wrapper.

Audit relevance:
Shell context interprets metacharacters, expansions, separators, and quoting.

Follow-up:
- verify shell avoidance or strict safe construction.

## J-S4. Expression or script source
Example idea:
- expression, rule, Groovy/SpEL snippet, or script body comes from request, config, database, or admin tool.

Audit relevance:
Expression sources may be code execution sources even without OS command APIs.

Follow-up:
- inspect engine restrictions, allowed features, and trusted-origin guarantees.

## J-S5. Queue/job/stored task source
Example idea:
- queued job argument, stored task definition, command metadata, or replay payload later launches a tool.

Audit relevance:
Stored and background values create second-order execution source paths.

Follow-up:
- identify writer path and revalidation before execution.

---

# 3. Case Templates

## Case J-S-CMD-1: Process argument source

Source focus:
Identify request or stored values that become `ProcessBuilder` argv entries or `Runtime.exec` arguments.

Recommended follow-up:
Verify fixed executable, strict argument allowlists, and option handling.

## Case J-S-CMD-2: Shell string source

Source focus:
Identify formatted command strings or templates passed to shell wrappers.

Recommended follow-up:
Verify whether untrusted content reaches shell context.

## Case J-S-CMD-3: Expression engine source

Source focus:
Identify expression/rule/script input and its origin before `ScriptEngineManager`, Groovy, SpEL, OGNL, MVEL, or JEXL evaluation.

Recommended follow-up:
Verify safe expression subsets and disabled dangerous features.

## Case J-S-CMD-4: External tool wrapper source

Source focus:
Identify tool selector, option values, file paths, environment values, and wrapper calls.

Recommended follow-up:
Trace wrapper internals and verify allowlist behavior.

---

# 4. Java-Specific Audit Heuristics

## 4.1 Request and DTO source heuristics
Pay attention to:
- `@PathVariable`
- `@RequestParam`
- `@RequestBody` DTO fields
- `@RequestHeader`
- multipart metadata
- admin/debug/diagnostic route parameters

## 4.2 Process API source heuristics
Pay attention to:
- `Runtime.getRuntime().exec(...)`
- `ProcessBuilder`
- values added to command lists
- dynamic executable paths
- environment maps and working directories
- helper wrappers returning `Process`

## 4.3 Shell source heuristics
Pay attention to:
- `sh -c`
- `bash -c`
- `cmd /c`
- `powershell -Command`
- string-built shell payloads
- generated script files

## 4.4 Expression and script source heuristics
Pay attention to:
- `ScriptEngineManager`
- Groovy execution
- SpEL / OGNL / MVEL / JEXL
- request-driven rule engines
- dynamic admin/debug evaluation features

## 4.5 External tool and job source heuristics
Pay attention to:
- file conversion tools
- network tools
- archive helpers
- shell scripts
- queue job arguments
- stored command templates
- wrappers that call CLIs indirectly

---

# 5. False-Positive Controls

Do not mark a Java source as high-priority if:
- the value is selected from a strict allowlist of safe commands, subcommands, or options,
- the executable is fixed and the source only affects non-dangerous display data,
- the value never reaches command construction, argv construction, expression evaluation, or execution wrappers,
- the stored value is trusted-only and cannot be attacker-influenced,
- expression input is fixed server-side and not externally influenced.

Use `Suspected source` or `Not enough evidence` if:
- wrapper helper behavior is hidden,
- expression engine configuration is not visible,
- queue/stored task writer paths are missing,
- shell vs argv context is unclear,
- allowlist behavior may exist elsewhere.

---

# 6. What Good Evidence Looks Like

Good Java source evidence includes:
- route/controller/worker/admin/import entry point,
- source annotation or API such as request param, DTO field, queue receive, config record, or stored task,
- propagation such as formatting, argv list assembly, shell template construction, environment map construction, or wrapper call,
- `Runtime.exec`, `ProcessBuilder`, script engine, expression engine, external tool wrapper, or job runner receiving the value,
- execution context when visible.

Good source evidence answers:
1. Which Java entry point receives the execution-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which command, shell, argv, expression, interpreter, or wrapper behavior should be audited next?
4. Is the source used for command selection, argument value, option, shell string, script, expression, environment, or job data?

---

# 7. Quick Java Source Checklist

- Are request values used as command arguments, flags, modes, or tool selectors?
- Are command names, scripts, subcommands, or interpreters dynamic?
- Are formatted strings passed to shell wrappers?
- Are expressions, rules, or scripts externally influenced?
- Are queue jobs or stored task definitions later executed?
- Are environment variables, working directories, config paths, or file paths influenced?
- Are wrapper helpers hiding the real execution boundary?
