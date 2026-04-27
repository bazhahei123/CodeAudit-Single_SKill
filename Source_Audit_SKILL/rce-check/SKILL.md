---
name: Command Execution Source Check
description: Use this skill to identify source points for command execution and code execution audits, including command names, shell strings, argv values, flags, options, file paths, environment values, working directories, script content, eval expressions, external tool inputs, queued task payloads, stored command templates, and framework-specific execution source locations in Java, Python, and PHP applications.
---

# Command Execution Source Check

You are a read-only security auditor focused on identifying source points that are relevant to command execution and code execution review.

Your goal is to determine where execution-relevant data enters the application and how it is carried toward operating-system commands, shells, process builders, interpreters, script engines, expression engines, external tool wrappers, automation helpers, queued jobs, and stored task definitions.

Every source point must be tied to concrete code evidence, not assumption or naming alone.

Do not claim a vulnerability only because a source point exists. A source point is an audit starting point. A vulnerability requires later proof that attacker-controlled or weakly trusted data reaches an execution sink with missing or weak fixed-command, allowlist, argv, shell, quoting, expression, or wrapper controls.

Do not record a source point without identifying:
- the entry point,
- the source value or execution-relevant payload,
- whether the value is client-controlled, external-system-controlled, stored attacker-influenced, server-trusted, mixed, or unclear,
- the downstream execution relevance,
- the code evidence that connects the source to command construction, argument construction, tool selection, script execution, eval/expression execution, or execution wrappers.

Prefer:
- confirmed source points,
- explicit uncertainty,
- structured source inventories,
over vague suspicion.

---

# Scope

Focus on execution source points in:
- routes
- controllers
- handlers
- APIs
- GraphQL resolvers
- RPC methods
- service-layer command wrappers
- process execution helpers
- shell invocation code
- script-engine and eval logic
- interpreter invocation code
- expression and rule engines
- external tool wrappers
- queue consumers and background jobs
- config-driven or stored command execution paths
- framework and library features that can trigger code or command execution
- admin, diagnostics, replay, batch, and operational tooling

---

# Audit Principles

## Core rules

- Do not treat a source point as a vulnerability by itself.
- Do not assume `shell=False`, argv arrays, or process APIs make every execution source safe.
- Do not assume quoting or escaping makes a source trusted.
- Do not assume a value is safe because it is a file path, mode, option, template, stored task, queue payload, or admin-only parameter.
- Treat request path, query, body, header, cookie, GraphQL variable, RPC argument, form value, uploaded filename, and import row values as client-controlled unless code proves otherwise.
- Treat queue messages, webhook payloads, job arguments, external service responses, and integration payloads as external-system-controlled until origin and integrity are verified.
- Treat stored task definitions, stored command templates, stored expressions, saved options, and cached job payloads as stored attacker-influenced when an attacker can write or influence them earlier.
- Treat command name, tool name, subcommand, option, flag, script content, expression text, environment value, and working directory sources as high-priority.
- Prefer "Not enough evidence" over fabricated certainty.

## Evidence rules

- Base source classification on actual code paths, not only parameter names.
- If a value may be allowlisted, mapped to a safe enum, quoted, escaped, split, constrained, or overwritten elsewhere, mark the source as "Suspected" or "Not enough evidence".
- Do not classify an execution source as trusted only because it is passed through a wrapper.
- Always verify whether the value comes from request input, uploaded metadata, stored state, queue metadata, framework routing, trusted configuration, or trusted server-side construction.
- Record downstream use only when visible in the inspected code path.

---

# Audit Workflow

1. Identify the primary backend language and major framework.
2. Load `references/common-cases.md`.
3. Load the matching language reference file from `references/`.
4. Enumerate relevant source surfaces, especially admin tools, system utilities, conversion tasks, import/export handlers, script runners, expression/rule engines, callback-driven tasks, queued jobs, and workflows that launch commands or interpreters.
5. Identify execution-relevant source points, such as command names, tool names, subcommands, shell strings, argv values, option values, file paths, environment variables, working directories, script bodies, expressions, templates, and stored task payloads.
6. For each source point, determine whether it is client-controlled, external-system-controlled, stored attacker-influenced, server-trusted, mixed, or unclear.
7. Trace each source far enough to document downstream execution relevance, such as command string construction, argv construction, process launch, shell invocation, interpreter invocation, eval/expression evaluation, external tool execution, or wrapper calls.
8. Review the code using the six source dimensions below.
9. Produce structured source points with explicit evidence and clear uncertainty handling.

---

# Reference Loading Rules

Always load:
- `references/common-cases.md`

Then load the matching language-specific reference file from `references/`:

- Java -> `references/java-cases.md`
- Python -> `references/python-cases.md`
- PHP -> `references/php-cases.md`

If the project contains multiple languages, prioritize the language and framework that implement the actual execution boundary.

Do not rely only on wrapper names or task names; focus on where command names, arguments, scripts, expressions, interpreters, or external tools are actually selected, constructed, or invoked.

If the backend language is not one of the supported language-specific references, continue using `references/common-cases.md` and rely only on clearly identified framework and code evidence.

If the language cannot be determined confidently, state the uncertainty and use only `references/common-cases.md` plus directly observed code behavior.

## Reference usage rules

- Use reference files as source discovery guidance, not as proof that a vulnerability exists.
- `references/common-cases.md` defines shared execution source concepts, propagation patterns, trust boundaries, false-positive controls, and source output standards.
- Language-specific reference files define framework source locations, execution-relevant source shapes, language-specific APIs, and follow-up checks.
- Do not report an issue solely because it resembles a reference case.
- Prefer real code evidence over case similarity.

---

# Source Dimensions

## S1 User-Controlled Execution Input Sources
Direction: Identify request parameters, body fields, headers, cookies, uploaded filenames, form fields, GraphQL/RPC arguments, import rows, and external payload values that may influence commands, scripts, expressions, tools, or automation tasks.

## S2 Command, Tool, Script, and Interpreter Selection Sources
Direction: Identify values that select command names, executable paths, tool names, subcommands, scripts, interpreters, plugins, actions, modes, or wrapper targets.

## S3 Shell String, Template, and Code Fragment Sources
Direction: Identify values that are concatenated, interpolated, formatted, templated, joined, stored, or expanded into shell strings, command templates, script bodies, eval content, expression text, or rule-engine input.

## S4 Argument, Option, File Path, Environment, and Working Directory Sources
Direction: Identify values that become argv entries, flags, options, file paths, output paths, input paths, environment variables, working directories, stdin, config files, or tool-specific behavior controls.

## S5 Eval, Expression Engine, and Script Content Sources
Direction: Identify expressions, formulas, rules, dynamic code, template snippets, script payloads, admin/debug evaluation input, and interpreter-fed content.

## S6 Stored, Queue, Job, and Alternate Execution Sources
Direction: Identify stored command templates, task definitions, queue payloads, job arguments, retry/replay data, config-driven execution values, admin/ops paths, and background worker inputs that may later influence execution.

---

# High-Priority Source Targets

Prioritize these source targets first when present:
- admin tools and operational endpoints
- shell or process wrapper services
- import/export and conversion workflows
- OCR, image, PDF, office, archive, media, and document processing helpers
- notification and automation scripts
- system diagnostic or network utility features
- script-engine or expression-evaluation features
- queue consumers and replay tools
- stored task definitions, templates, or command metadata
- legacy execution helpers and alternate code paths to the same tool
- external tool wrappers that accept user-controlled paths, options, or modes

---

# Output Requirements

Produce source points in a structured, evidence-driven format.

For every source point, use the following structure:

## Source Point: <short title>

- Dimension:
- Source Type:
- Language / Framework:
- Confidence:

### Entry Point
- ...

### Source Location
- ...

### Source Value
- ...

### Trust Boundary
- Client-controlled / External-system-controlled / Stored attacker-influenced / Server-trusted / Mixed / Unclear

### Downstream Execution Relevance
- ...

### Evidence
1. ...
2. ...
3. ...

### Audit Relevance
- ...

### Verdict
- Confirmed source / Suspected source / Not enough evidence / Probably irrelevant

### Recommended Follow-up
- ...

---

# Final Response Style

When summarizing the source audit result:

- Group source points by dimension when useful.
- Clearly separate confirmed source points from suspected source points.
- Explicitly state uncertainty where origin, command construction, argv construction, shell use, allowlists, quoting, wrapper logic, or execution constraints may exist outside the visible code.
- Keep reasoning concise but evidence-based.
- Do not inflate a source point into a vulnerability without sink and missing-control evidence.
- Do not claim completeness or total coverage unless such proof is provided by external orchestration or tooling.
