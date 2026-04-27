# PHP Command Execution Source Cases

## Purpose

This file contains PHP-specific source point patterns and audit cases for command execution and code execution source discovery.

Use it when the target application is primarily implemented in PHP, especially in:
- Laravel
- Symfony
- raw PHP applications
- `system`, `exec`, `shell_exec`, `passthru`
- `proc_open`, `popen`
- `eval`
- PHP backends exposing admin tools, automation, conversion, or external-tool workflows

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream execution behavior in the target code.

---

# 1. PHP Source Discovery Points

Prioritize these source values and events:
- `$_GET`, `$_POST`, `$_REQUEST`, route parameters, request input, headers, cookies, and uploaded metadata
- command names, tool names, shell script names, subcommands, actions, and modes
- shell command strings, command templates, and generated PHP/script fragments
- option values, flags, file paths, URLs, environment variables, working directories, and stdin/config values
- eval input, generated PHP code, expression strings, and admin/debug evaluation payloads
- stored command templates, queue payloads, job arguments, cron/admin task definitions, and replay data

Source questions:
- Which source supplies command name, shell string, option, file path, eval code, or tool target?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value concatenated, escaped, quoted, allowlisted, stored, or passed through a wrapper before use?
- Which execution context should be audited next: shell, process wrapper, eval, external tool, or background job?

---

# 2. PHP Source Patterns

## H-S1. Request-derived shell argument
Example idea:
- request value such as host, path, URL, file, mode, or target is concatenated into a shell command or passed to a wrapper.

Audit relevance:
The source may affect shell parsing, tool behavior, option parsing, or file access.

Follow-up:
- trace into `system`, `exec`, `shell_exec`, `passthru`, `proc_open`, `popen`, or wrapper helpers.

## H-S2. Dynamic command or tool selector
Example idea:
- request or stored value selects command name, script, tool, action, or mode.

Audit relevance:
Command/tool selection sources are high priority even if arguments are separately escaped.

Follow-up:
- verify fixed commands or strict allowlists.

## H-S3. Shell command template source
Example idea:
- formatted string, stored command template, or generated shell script includes request or stored values.

Audit relevance:
Shell strings interpret metacharacters, separators, expansions, and quoting.

Follow-up:
- verify shell avoidance or strict safe construction.

## H-S4. Eval or generated code source
Example idea:
- PHP code, expression text, template snippet, or admin/debug payload is passed toward `eval` or dynamic execution helper.

Audit relevance:
Eval sources may be code execution sources even without OS command APIs.

Follow-up:
- verify trusted origin or strict safe expression subsets.

## H-S5. Queue/job/stored task source
Example idea:
- queue payload, stored task definition, cron metadata, or replay data later launches a CLI or eval path.

Audit relevance:
Stored and background values create second-order execution source paths.

Follow-up:
- identify writer path and revalidation before execution.

---

# 3. Case Templates

## Case H-S-CMD-1: Shell argument source

Source focus:
Identify request or stored values that become shell command fragments or tool arguments.

Recommended follow-up:
Verify fixed executable, shell avoidance, allowlists, and option handling.

## Case H-S-CMD-2: Dynamic command source

Source focus:
Identify command/tool/script selector values and their origin.

Recommended follow-up:
Verify command selection is fixed or strictly allowlisted.

## Case H-S-CMD-3: Eval source

Source focus:
Identify PHP code, expression, formula, template snippet, or debug/admin input passed toward eval-like behavior.

Recommended follow-up:
Verify no attacker-influenced code reaches dynamic execution.

## Case H-S-CMD-4: Background tool source

Source focus:
Identify queue payload, stored command template, cron/admin task, and wrapper arguments.

Recommended follow-up:
Trace writer and worker paths and verify second-order execution controls.

---

# 4. PHP-Specific Audit Heuristics

## 4.1 Request and upload source heuristics
Pay attention to:
- `$_GET`
- `$_POST`
- `$request->input(...)`
- route parameters
- uploaded filenames or metadata
- admin/debug route inputs

## 4.2 Process API source heuristics
Pay attention to:
- `system`
- `exec`
- `shell_exec`
- `passthru`
- backticks
- `proc_open`
- `popen`
- values passed into wrapper helpers

## 4.3 Shell and option source heuristics
Pay attention to:
- concatenated shell strings
- dynamic command names
- user-controlled option strings
- shell script invocation
- helper methods that claim to sanitize commands
- file paths and config paths passed to tools

## 4.4 Eval and dynamic execution source heuristics
Pay attention to:
- `eval`
- generated PHP code
- dynamic include/execute helpers
- admin/debug evaluation features
- expression-like wrappers

## 4.5 External tool and job source heuristics
Pay attention to:
- image/PDF/archive/media conversion tools
- shell scripts
- cron/admin operational helpers
- queue jobs and replay tools
- stored task definitions

---

# 5. False-Positive Controls

Do not mark a PHP source as high-priority if:
- the value is selected from a strict allowlist of safe commands, subcommands, or options,
- the executable is fixed and the source only affects non-dangerous display data,
- the value never reaches command construction, shell execution, eval, or execution wrappers,
- the stored value is trusted-only and cannot be attacker-influenced,
- eval input is fixed server-side and not externally influenced.

Use `Suspected source` or `Not enough evidence` if:
- wrapper helper behavior is hidden,
- escaping or allowlist behavior is not visible,
- queue/stored task writer paths are missing,
- shell vs process context is unclear,
- eval-like behavior may exist behind a helper.

---

# 6. What Good Evidence Looks Like

Good PHP source evidence includes:
- route/controller/script/worker/admin/import entry point,
- source API such as `$_GET`, request input, uploaded metadata, queue payload, config record, or stored task,
- propagation such as concatenation, interpolation, escaping, quoting, storage, or wrapper call,
- command API, eval helper, external tool wrapper, cron helper, or job runner receiving the value,
- execution context when visible.

Good source evidence answers:
1. Which PHP entry point receives the execution-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which shell, process, eval, external tool, or wrapper behavior should be audited next?
4. Is the source used for command selection, argument value, option, shell string, script, expression, environment, or job data?

---

# 7. Quick PHP Source Checklist

- Are request values used as command arguments, flags, modes, or tool selectors?
- Are command names, scripts, subcommands, or tool choices dynamic?
- Are strings passed to `system`, `exec`, `shell_exec`, `passthru`, `proc_open`, or backticks?
- Are eval payloads, generated code, or expression strings externally influenced?
- Are queue jobs or stored task definitions later executed?
- Are environment variables, working directories, config paths, or file paths influenced?
- Are wrapper helpers hiding the real execution boundary?
