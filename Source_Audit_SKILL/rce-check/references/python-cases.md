# Python Command Execution Source Cases

## Purpose

This file contains Python-specific source point patterns and audit cases for command execution and code execution source discovery.

Use it when the target application is primarily implemented in Python, especially in:
- Django
- Flask
- FastAPI
- `os.system`
- `os.popen`
- `subprocess`
- `eval` / `exec`
- Python backends exposing admin tools, conversion, automation, or external-tool workflows

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream execution behavior in the target code.

---

# 1. Python Source Discovery Points

Prioritize these source values and events:
- route parameters, query values, form fields, JSON body fields, headers, cookies, and uploaded metadata
- command names, tool names, script paths, subcommands, actions, and modes
- string-form subprocess commands, shell strings, command templates, and generated scripts
- argv list entries, option values, flags, file paths, URLs, environment variables, working directories, stdin, and config paths
- eval/exec input, expression strings, formulas, dynamic imports, and admin/debug evaluation payloads
- stored command templates, Celery/task payloads, queue job arguments, scheduled task definitions, and replay data

Source questions:
- Which source supplies command name, argv value, shell string, option, file path, eval code, or tool target?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value formatted, split, quoted, allowlisted, stored, or passed through a wrapper before use?
- Which execution context should be audited next: shell, argv, eval/exec, interpreter, external tool, or background job?

---

# 2. High-Coverage Python Source Candidate Inventory

Use these candidate lists to seed graph queries and text searches. Keep a candidate only when the code shows command construction, argv construction, shell interpretation, eval/exec, dynamic import, interpreter invocation, external tool execution, or stored/background execution relevance.

## 2.1 Web, API, and request entry candidates

- Django URL `path(...)`
- Django `re_path(...)`
- Django views
- DRF `APIView`
- DRF `ViewSet`
- DRF `GenericViewSet`
- DRF `ModelViewSet`
- DRF `@api_view`
- DRF `@action`
- Flask `@app.route`
- Flask `@blueprint.route`
- FastAPI `@app.get`
- FastAPI `@app.post`
- FastAPI `@app.put`
- FastAPI `@app.delete`
- FastAPI `APIRouter`
- Starlette `Route`
- Tornado `RequestHandler`
- GraphQL resolver
- GraphQL mutation
- gRPC servicer method
- Celery task
- RQ job
- Dramatiq actor
- webhook handler
- admin command

## 2.2 Request, task, and payload source candidates

- `request.GET`
- `request.POST`
- `request.data`
- `request.query_params`
- `request.args`
- `request.form`
- `request.json`
- `request.get_json()`
- `request.headers`
- `request.cookies`
- FastAPI `Path`
- FastAPI `Query`
- FastAPI `Body`
- `UploadFile.filename`
- Celery task args
- queue message body
- Pydantic fields
- GraphQL arguments

## 2.3 Command, tool, script, and action selector candidates

- `cmd`
- `command`
- `command_name`
- `tool`
- `tool_name`
- `executable`
- `program`
- `binary`
- `process`
- `script`
- `script_name`
- `script_path`
- `interpreter`
- `runtime`
- `engine`
- `subcommand`
- `action`
- `operation`
- `mode`
- `runner`
- `plugin`
- `task_name`
- `job_type`

## 2.4 Argument, option, path, environment, and cwd candidates

- `arg`
- `args`
- `argv`
- `arguments`
- `option`
- `options`
- `flag`
- `flags`
- `target`
- `host`
- `ip`
- `domain`
- `url`
- `file`
- `filename`
- `path`
- `input`
- `output`
- `config`
- `env`
- `environment`
- `cwd`
- `work_dir`
- `working_dir`
- `stdin`
- `payload`
- `timeout`
- `profile`

## 2.5 Shell, eval, dynamic import, and script candidates

- `shell`
- `shell_cmd`
- `shell_command`
- `command_line`
- `command_template`
- `f-string`
- `.format`
- `template`
- `script_body`
- `source_code`
- `code`
- `expression`
- `expr`
- `formula`
- `rule`
- `eval_input`
- `debug_code`
- `module`
- `module_name`
- `callable`
- `function_name`
- `import_path`

## 2.6 Downstream execution mapping candidates

- `subprocess.run`
- `subprocess.call`
- `subprocess.Popen`
- `subprocess.check_output`
- `subprocess.check_call`
- `os.system`
- `os.popen`
- `commands.getoutput`
- `pty.spawn`
- `asyncio.create_subprocess_exec`
- `asyncio.create_subprocess_shell`
- `shell=True`
- `eval`
- `exec`
- `compile`
- `__import__`
- `importlib.import_module`
- `getattr`
- `jinja2.Template`
- external tool wrappers
- `ffmpeg`
- `convert`
- `pandoc`
- `wkhtmltopdf`

## 2.7 Python graph search recipes

```text
route/view/@app.post + request args/body cmd/tool/action + subprocess/os.system
request.data + args/options/file/path + subprocess list argv
f-string/format/template + command_template/shell_command + shell=True/os.system
eval/exec/compile/importlib + expression/code/module/callable + request/stored value
Celery/RQ/Dramatiq + task args/queue payload + subprocess/wrapper
env/cwd/stdin/config + request/task/stored value + subprocess.Popen
```

---

# 3. Python Source Patterns

## P-S1. Request-derived command argument
Example idea:
- request value such as host, file, URL, mode, or target becomes a subprocess argument or shell fragment.

Audit relevance:
The source may affect command behavior, tool target, option parsing, or file access.

Follow-up:
- trace into `subprocess`, `os.system`, `os.popen`, or wrapper helpers.

## P-S2. Dynamic command or tool selector
Example idea:
- request or stored field selects `tool_name`, script, command, subcommand, interpreter, or action.

Audit relevance:
Command/tool selection sources are high priority even when `shell=False`.

Follow-up:
- verify fixed commands or strict allowlists.

## P-S3. Shell string or command template source
Example idea:
- f-string, formatted command, stored task command, or generated shell script includes request or stored values.

Audit relevance:
Shell context interprets metacharacters, substitutions, separators, expansions, and quoting.

Follow-up:
- verify whether source reaches `shell=True`, `os.system`, shell scripts, or string-form wrappers.

## P-S4. Eval, exec, or dynamic import source
Example idea:
- expression, formula, Python code, module name, callable name, or debug/admin payload is passed toward dynamic execution.

Audit relevance:
These sources may be code execution sources even without OS command APIs.

Follow-up:
- verify safe expression subsets, import allowlists, and disabled dangerous features.

## P-S5. Queue/job/stored task source
Example idea:
- Celery task argument, queue payload, stored command template, scheduler metadata, or replay payload later launches a tool.

Audit relevance:
Stored and background values create second-order execution source paths.

Follow-up:
- identify writer path and revalidation before execution.

---

# 4. Case Templates

## Case P-S-CMD-1: Subprocess argument source

Source focus:
Identify request or stored values that become subprocess argv entries, tool targets, options, or file paths.

Recommended follow-up:
Verify fixed executable, strict argument allowlists, and option handling.

## Case P-S-CMD-2: Shell string source

Source focus:
Identify formatted command strings or templates passed to `shell=True`, `os.system`, or shell wrappers.

Recommended follow-up:
Verify whether untrusted content reaches shell context.

## Case P-S-CMD-3: Eval/exec source

Source focus:
Identify expression, Python code, formula, module name, callable name, or debug/admin input passed toward dynamic execution.

Recommended follow-up:
Verify safe subsets, allowlists, and trusted-origin guarantees.

## Case P-S-CMD-4: Worker tool source

Source focus:
Identify Celery/task payload, stored command template, queue job data, and wrapper arguments.

Recommended follow-up:
Trace writer and worker paths and verify second-order execution controls.

---

# 5. Python-Specific Audit Heuristics

## 4.1 Request and upload source heuristics
Pay attention to:
- Django/Flask/FastAPI request args, form, JSON, headers, and cookies
- route parameters
- uploaded filenames or metadata
- admin/debug route inputs
- import/export parameters

## 4.2 Process API source heuristics
Pay attention to:
- `os.system`
- `os.popen`
- `subprocess.run`
- `subprocess.call`
- `subprocess.Popen`
- command lists and string commands
- wrapper helper inputs

## 4.3 Shell and option source heuristics
Pay attention to:
- `shell=True`
- string-form subprocess commands
- shell script invocation
- user-controlled content passed through shell wrappers
- option-like values starting with `-`
- file paths and config paths passed to tools

## 4.4 Eval and dynamic execution source heuristics
Pay attention to:
- `eval`
- `exec`
- dynamically executed expressions
- dynamic imports
- admin/debug features evaluating Python expressions
- rule-engine style wrappers

## 4.5 External tool and worker source heuristics
Pay attention to:
- OCR or conversion helpers
- archive or media-processing tools
- automation scripts
- queued workers launching CLIs
- retry/replay paths that relaunch tools
- stored task definitions

---

# 6. False-Positive Controls

Do not mark a Python source as high-priority if:
- the value is selected from a strict allowlist of safe commands, subcommands, or options,
- the executable is fixed and the source only affects non-dangerous display data,
- the value never reaches command construction, argv construction, eval/exec, or execution wrappers,
- the stored value is trusted-only and cannot be attacker-influenced,
- eval/exec input is fixed server-side and not externally influenced.

Use `Suspected source` or `Not enough evidence` if:
- wrapper helper behavior is hidden,
- `shell=True` vs argv context is unclear,
- queue/stored task writer paths are missing,
- safe expression or import allowlists may exist elsewhere,
- option handling is tool-specific and not visible.

---

# 7. What Good Evidence Looks Like

Good Python source evidence includes:
- route/view/worker/admin/import entry point,
- source API such as request input, uploaded metadata, queue payload, task argument, config record, or stored task,
- propagation such as f-string formatting, argv list assembly, shell template construction, environment map construction, storage, or wrapper call,
- `subprocess`, `os.system`, `os.popen`, eval/exec, external tool wrapper, or job runner receiving the value,
- execution context when visible.

Good source evidence answers:
1. Which Python entry point receives the execution-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which shell, argv, eval/exec, external tool, or wrapper behavior should be audited next?
4. Is the source used for command selection, argument value, option, shell string, script, expression, environment, or job data?

---

# 8. Quick Python Source Checklist

- Are request values used as command arguments, flags, modes, or tool selectors?
- Are command names, scripts, subcommands, interpreters, or tool choices dynamic?
- Are formatted strings passed to `shell=True`, `os.system`, shell scripts, or wrappers?
- Are eval/exec payloads, generated code, dynamic imports, or expression strings externally influenced?
- Are queue jobs or stored task definitions later executed?
- Are environment variables, working directories, config paths, or file paths influenced?
- Are wrapper helpers hiding the real execution boundary?
