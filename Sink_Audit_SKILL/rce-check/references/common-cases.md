# Command Execution Common Cases

## Purpose

This file defines shared command execution and code execution concepts, audit logic, anti-patterns, false-positive controls, and finding standards that apply across languages and frameworks.

Use this file as the base reference for command execution review before loading any language-specific reference.

This file explains:
- what counts as an execution boundary,
- what counts as a source, sink, context, and control,
- how to distinguish command injection, shell injection, argument injection, option injection, and eval-like execution,
- why shell context and argv context differ,
- when to report `Confirmed`, `Suspected`, or `Not enough evidence`.

This reference is guidance, not proof. Do not report a vulnerability only because code resembles a pattern described here. Always verify the real data flow, real execution sink, and real execution context in the target code.

---

# 1. Core Concepts

## 1.1 What command execution risk is
Command execution risk exists when attacker-controlled or weakly trusted input can influence operating-system command execution, shell interpretation, interpreter invocation, expression evaluation, or dangerous external tool execution.

The core question is:

**Can attacker-controlled input influence what program, script, shell, expression engine, or external tool is executed, or how it is executed?**

## 1.2 Source, propagation, sink, and execution context

### Source
A source is any attacker-controllable input, including:
- query parameters
- path parameters
- request body fields
- headers
- cookies
- uploaded filenames or metadata
- stored values previously written by a user or admin
- queue payloads
- config values influenced by external actors
- API responses derived from user-controlled content

### Propagation
Propagation means the input is copied, normalized, concatenated, formatted, wrapped, stored, or transformed before reaching an execution sink.

Do not assume a renamed or "validated" variable is safe unless the constraint is actually enforceable and visible.

### Sink
A sink is the place where code, commands, or tools are executed, including:
- OS command APIs
- shell wrappers
- process builders
- eval or exec-style APIs
- expression engines
- script engines
- interpreter invocations
- external tool wrappers
- automation or helper services that launch programs

### Execution context
Execution context describes how the sink interprets input.

Examples:
- shell context
- argv / argument-array context
- interpreter context
- eval / expression context
- external tool context

A finding is often only meaningful when the execution context is clear.

---

# 2. Shared Attack Surfaces

Prioritize these attack surfaces first:

- admin and operational tools
- public route/controller annotations and HTTP handler registration
- GraphQL mutation or resolver handlers that trigger tools, scripts, reports, or jobs
- RPC, gRPC, SOAP, WCF, Thrift, Binder, DBus, or internal protocol methods that launch processes or evaluate expressions
- WebSocket and message-frame handlers carrying command, script, task, or tool selectors
- diagnostics or network utility features
- file conversion and media processing workflows
- import/export paths
- OCR, PDF, office, image, archive, or document helpers
- wrapper services for shell/process execution
- queue consumers and background tasks
- webhook, callback, replay, and reconciliation handlers that run external tools
- mobile exported component, deep-link, WebView bridge, SDK callback, or IPC entry points
- native service socket, pipe, IPC, or custom binary protocol handlers that launch tools
- script-runner or automation features
- expression or template evaluation helpers
- legacy helpers and alternate execution paths

---

# 2.1 Candidate search groups for graph workflows

When a graph database cannot start from a single universal sink, build candidate sets from four groups and intersect them.

### Entry candidates
Search for externally reachable or semi-trusted execution points:
- controller
- route
- handler
- endpoint
- resolver
- mutation
- action
- servlet
- filter
- interceptor
- middleware
- webhook
- callback
- listener
- consumer
- subscriber
- receiver
- worker
- job
- scheduler
- admin
- ops
- debug
- diagnostics
- ping
- traceroute
- nslookup
- curl
- wget
- import
- export
- convert
- render
- preview
- generate
- scan
- OCR
- PDF
- office
- media
- archive
- backup
- restore
- replay
- script
- task
- command
- shell
- terminal
- IPC
- RPC
- WebSocket
- deep link
- exported
- bridge

### Execution sink candidates
Search for code, process, shell, interpreter, or external tool execution:
- exec
- execute
- run
- start
- spawn
- system
- shell
- command
- cmd
- process
- ProcessBuilder
- Runtime.exec
- subprocess
- popen
- proc_open
- eval
- exec code
- compile
- script
- interpreter
- expression
- template eval
- rule eval
- dynamic import
- class load
- assembly load
- library load
- plugin load
- reflection invoke
- powershell
- bash
- sh -c
- cmd /c
- python -c
- node -e
- php -r
- groovy
- perl
- ruby
- lua
- ffmpeg
- imagemagick
- convert
- ghostscript
- libreoffice
- pandoc
- tar
- zip
- 7z
- git
- ssh
- curl
- wget
- ping
- nslookup

### Command construction candidates
Search for untrusted values influencing executable strings, argv arrays, tool names, flags, scripts, or environment:
- command template
- cmdline
- toolName
- executable
- binary
- scriptPath
- script
- args
- argv
- option
- flag
- mode
- format
- profile
- preset
- file path
- environment
- working directory
- concatenation
- interpolation
- formatted string
- join args
- split command
- quote
- escape
- sanitize
- base64 decode
- URL decode
- stored task
- scheduled task
- retry command
- replay command

### Required-control candidates
Search near candidate sinks for controls:
- fixed command
- command allowlist
- tool allowlist
- option allowlist
- allowed commands
- allowed tools
- allowed args
- argv array
- shell disabled
- shell=false
- UseShellExecute=false
- no shell
- validate
- reject
- enum
- schema
- DTO
- timeout
- kill
- resource limit
- max output
- working directory allowlist
- environment allowlist
- sandbox
- seccomp
- chroot
- container
- non-root
- least privilege
- safe expression
- restricted globals
- script disabled
- language subset
- signature
- trusted-only
- audit log

## 2.2 Generic graph search recipes

Useful candidate recipes:

```text
<entry candidate> + <execution sink candidate>
<admin/diagnostic/import/export/convert/render> + <process/shell sink>
<request/body/header/cookie/file/job payload> + <command construction candidate>
<toolName/args/option/script> + <run/exec/start/spawn>
<eval/expression/template/rule> + <request/stored value>
<queue/job/replay> + <shell/process/eval sink>
<external tool> + <user-controlled path/option/preset>
<candidate sink> without nearby <required-control candidate>
<argv mode> + <user-controlled command/tool/option>
<shell context> + <concatenation/interpolation/template>
```

---

# 3. Shared Risk Categories

## 3.1 Command injection
User input changes the command string or shell command in a dangerous way.

Examples:
- concatenated shell command string
- `ping ` + user input
- raw shell script fragment with attacker-controlled value

## 3.2 Shell injection
User input reaches a shell interpreter such as:
- `sh -c`
- `bash -c`
- `cmd /c`
- `powershell -Command`

Why important:
The shell may interpret separators, substitutions, expansions, quoting edge cases, and operators.

## 3.3 Argument injection
User input does not necessarily break out into a separate command, but still changes the behavior of the executed program through arguments.

Examples:
- attacker controls option-like values
- attacker injects file paths, flags, or mode selectors
- attacker controls command name selection

## 3.4 Option injection
User input is interpreted as a command-line option or flag by the target tool.

Examples:
- attacker input begins with `-`
- value becomes a dangerous flag or config override
- file path or user data is parsed as a program option

## 3.5 Eval / expression execution
User input reaches:
- `eval`
- `exec`
- script engine
- expression engine
- dynamic rule engine
- runtime code generation

Why important:
This may be code execution even without OS shell involvement.

## 3.6 Indirect execution through wrappers
The code does not call execution APIs directly, but goes through wrappers such as:
- `runCommand(...)`
- `invokeTool(...)`
- `executeTask(...)`
- `renderExpression(...)`

Why important:
Real projects often hide dangerous sinks behind helper layers.

---

# 4. Shell Context vs argv Context

## 4.1 Shell context
In shell context, input may be interpreted by a shell.

Typical danger:
- separators
- quoting confusion
- substitutions
- shell metacharacters
- environment and expansion behavior

Shell context is generally higher risk.

## 4.2 argv context
In argv context, the program receives an argument array directly.

Typical danger:
- command name selection
- option injection
- dangerous path or file arguments
- tool-specific behavior changes

argv context is often safer than shell context, but it is not automatically safe.

## 4.3 Why this matters
The same user-controlled value may be:
- catastrophic in shell context,
- partially dangerous in argv context,
- or safe only if fixed command and allowlisted arguments are enforced.

Do not treat all execution APIs as equivalent.

---

# 5. Shared Anti-Patterns

These are common danger signals across languages.

## A1. Concatenated command strings
High-risk pattern:
- building a command string with string concatenation, interpolation, formatting, or templates

Why risky:
Attacker input may directly alter command behavior.

## A2. Shell wrappers with user-controlled content
High-risk pattern:
- shell command helpers
- `sh -c`, `bash -c`, `cmd /c`, `powershell -Command`
- interpreter invoked with attacker-influenced script fragment

Why risky:
The shell or interpreter performs extra parsing.

## A3. Dynamic command or tool selection
High-risk pattern:
- command name chosen by request field
- tool name selected dynamically
- subcommand or script chosen by user input

Why risky:
Even if arguments are separated safely, attacker control of what program runs is dangerous.

## A4. Option and argument injection
High-risk pattern:
- user input passed directly as tool option
- user input can become `-flag`
- no fixed allowlist for allowed arguments

Why risky:
Tool behavior may be hijacked without breaking out into a second command.

## A5. Unsafe eval or expression execution
High-risk pattern:
- `eval`
- `exec`
- dynamic expression evaluation
- rule engine / script engine with attacker-influenced content

Why risky:
The attacker may execute code or dangerous expressions directly.

## A6. Wrapper abstraction hiding the real sink
High-risk pattern:
- helper name looks harmless
- helper eventually reaches system, process, shell, or eval sink

Why risky:
Shallow grep may miss the real boundary.

## A7. Stored or second-order execution
High-risk pattern:
- attacker input stored first
- later reused in command template, scheduled task, or tool invocation

Why risky:
The dangerous execution path may be delayed and overlooked.

## A8. False confidence from quoting
High-risk pattern:
- one quoting helper used as sole protection
- escaping assumptions without verifying shell/tool semantics

Why risky:
Quoting is context-sensitive and often incomplete.

---

# 6. Shared Protection Model

## 6.1 What strong protection usually looks like

Strong protections typically include:
- fixed command names
- fixed command arrays instead of shell strings
- no shell invocation for untrusted input
- strict allowlists for allowed tools, subcommands, and options
- validation of file paths, modes, and flags
- no eval-style execution on attacker-controlled content
- safe expression subsets or disabled dangerous features
- wrapper services that enforce trusted-only execution contracts

## 6.2 What does not automatically mean the code is safe

The following do **not** automatically mean the code is safe:
- using a process API instead of a shell string
- setting `shell=False` while still trusting dangerous arguments
- quoting user input once
- using a helper or wrapper service
- using an expression engine with "simple" expressions
- running commands only in background jobs
- calling internal tools only
- validating only with regex if the command, option set, or execution path is still too broad

---

# 7. False-Positive Controls

Do not report a vulnerability as `Confirmed` if:
- the input is provably constant or server-controlled,
- the executed command, subcommand, and options are all fixed and not attacker-influenced,
- the sink receives only allowlisted argument values with no shell interpretation,
- the apparent sink is unreachable from untrusted input,
- the wrapper clearly enforces a narrow trusted execution contract,
- the expression engine evaluates only a fixed safe expression set with no attacker control.

Use `Suspected` or `Not enough evidence` if:
- the sink exists but attacker influence on the input is unclear,
- user input reaches a wrapper, but the real execution boundary is hidden,
- quoting, allowlisting, or parser behavior may exist elsewhere but cannot be verified,
- an external tool is dangerous but actual argument control is not visible.

Do not over-claim based only on:
- the presence of process APIs,
- the presence of helper names like `run`, `exec`, or `invoke`,
- the use of `shell=False` or equivalent by itself,
- the existence of `eval` without attacker influence,
- stored templates or jobs without proving attacker control.

---

# 8. Finding Classification

## Confirmed
Use `Confirmed` when there is clear evidence that:
- attacker-controlled or weakly trusted input reaches an execution sink,
- protection is absent, weak, or bypassable,
- and dangerous command, code, shell, or interpreter influence is plausibly achievable.

## Suspected
Use `Suspected` when:
- there is a high-risk source-to-sink pattern,
- a shell or eval boundary appears reachable,
- but exact allowlist, quoting, or wrapper behavior remains hidden.

## Not enough evidence
Use `Not enough evidence` when:
- the sink is visible but the input source is not,
- the source is visible but the true execution boundary is hidden,
- or tool / wrapper semantics cannot be verified from available code.

## Probably safe
Use `Probably safe` when:
- the path is visible,
- execution is constrained to fixed commands or strongly allowlisted arguments,
- and no shell / eval / dangerous wrapper expansion is evident.

---

# 9. What Good Evidence Looks Like

Strong command execution findings usually include:
- the exact entry point
- the tainted input source
- the propagation path
- the execution sink
- the execution context
- the missing or weak allowlist / safe-construction control
- the reason attacker input can alter executed behavior

Good findings usually answer:
1. What input is attacker-controlled?
2. Where does it travel?
3. What execution boundary does it reach?
4. Is the context shell, argv, eval, interpreter, or tool wrapper?
5. What protection should have prevented dangerous influence?

---

# 10. Shared Remediation Guidance

Preferred fixes include:
- replace shell string construction with fixed argv arrays when possible
- keep command names fixed in code
- allowlist subcommands, flags, and modes strictly
- reject or normalize dangerous option-like user input where appropriate
- avoid eval / exec on attacker-influenced content
- restrict expression engines to safe subsets
- move dangerous tool execution behind narrow trusted wrappers
- treat stored task definitions and queued actions as untrusted unless validated

Avoid weak fixes such as:
- quoting alone
- blacklist filtering only
- relying on "internal use" assumptions
- trusting background jobs more than request paths
- keeping shell execution while only lightly sanitizing inputs

---

# 11. Shared Quick Checklist

Use this as a reminder, not as a substitute for reasoning.

- Is there a real attacker-controlled or weakly trusted source?
- Does it reach a command, shell, eval, interpreter, or tool sink?
- Is the execution context shell, argv, eval, interpreter, or wrapper-based?
- Can the attacker influence command name, option, path, or script content?
- Is there a strict allowlist?
- Is shell interpretation avoided where possible?
- Can stored or queued data later influence execution?
- Are alternate paths to the same tool equally constrained?
