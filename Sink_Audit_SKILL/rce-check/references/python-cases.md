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

# 2. High-Coverage Python Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, execution context, sink behavior, and missing controls.

## 2.1 Web, API, and request entry candidates
Search for:
- `@app.route`
- `@blueprint.route`
- `@bp.route`
- `Flask`
- `request.args`
- `request.form`
- `request.values`
- `request.files`
- `request.cookies`
- `request.headers`
- `request.get_json`
- `request.data`
- `@api_view`
- `APIView`
- `ViewSet`
- `ModelViewSet`
- `GenericAPIView`
- `path(`
- `re_path(`
- `View`
- `dispatch`
- `get`
- `post`
- `put`
- `patch`
- `delete`
- `@csrf_exempt`
- `@app.get`
- `@app.post`
- `FastAPI`
- `APIRouter`
- `UploadFile`
- `File`
- `Form`
- `Query`
- `Path`
- `Header`
- `Cookie`
- `Request`
- `GraphQLView`
- `Mutation`

## 2.2 Worker, message, import/export, and admin entries
Search for:
- `@shared_task`
- `@app.task`
- `Celery`
- `dramatiq.actor`
- `rq.job`
- `huey.task`
- `BaseCommand`
- `handle`
- `KafkaConsumer`
- `pika`
- `basic_consume`
- `on_message`
- `consume`
- `websocket`
- `admin`
- `debug`
- `diagnostic`
- `convert`
- `render`
- `import`
- `export`
- `scan`
- `preview`
- `generate`
- `replay`
- `run_task`
- `execute_task`

## 2.3 Process, shell, and OS command sink candidates
Search for:
- `os.system`
- `os.popen`
- `os.spawn`
- `os.spawnl`
- `os.spawnlp`
- `os.spawnv`
- `os.spawnvp`
- `os.exec`
- `os.execv`
- `os.execve`
- `os.execl`
- `subprocess.run`
- `subprocess.call`
- `subprocess.check_call`
- `subprocess.check_output`
- `subprocess.Popen`
- `asyncio.create_subprocess_exec`
- `asyncio.create_subprocess_shell`
- `commands.getoutput`
- `commands.getstatusoutput`
- `pty.spawn`
- `shlex.split`
- `shell=True`
- `/bin/sh`
- `bash -c`
- `cmd.exe`
- `powershell`
- `plumbum`
- `sh.Command`
- `fabric.Connection.run`
- `invoke.run`
- `pexpect.spawn`

## 2.4 Eval, interpreter, template, and dynamic code sink candidates
Search for:
- `eval`
- `exec`
- `compile`
- `execfile`
- `input(`
- `ast.parse`
- `literal_eval` review
- `importlib.import_module`
- `__import__`
- `getattr`
- `setattr`
- `globals()`
- `locals()`
- `runpy.run_path`
- `runpy.run_module`
- `code.InteractiveInterpreter`
- `IPython`
- `jinja2.Template`
- `Template.render`
- `render_template_string`
- `mako.template.Template`
- `yaml.load` with unsafe tags
- `pickle.loads` when used as code loading path
- `ctypes.CDLL`
- `ctypes.cdll.LoadLibrary`

## 2.5 External tool and converter sink candidates
Search for wrappers or command names around:
- `ffmpeg`
- `magick`
- `convert`
- `identify`
- `gs`
- `ghostscript`
- `pdftotext`
- `pdfinfo`
- `wkhtmltopdf`
- `libreoffice`
- `soffice`
- `pandoc`
- `tesseract`
- `tar`
- `zip`
- `unzip`
- `7z`
- `git`
- `ssh`
- `scp`
- `curl`
- `wget`
- `ping`
- `traceroute`
- `nslookup`
- `openssl`
- `docker`
- `kubectl`

## 2.6 Command and argument construction candidates
Search for:
- `cmd`
- `command`
- `args`
- `argv`
- `tool_name`
- `script_path`
- `shell_cmd`
- `command_template`
- `f"{"`
- `.format(`
- `%`
- `" ".join`
- `split(" ")`
- `shlex.quote`
- `shlex.split`
- `quote`
- `escape`
- `sanitize`
- `base64.b64decode`
- `urllib.parse.unquote`
- `cwd=`
- `env=`
- `executable=`
- `stdin=`
- `timeout=`
- `allowed_commands`
- `allowed_tools`

## 2.7 Required-control candidates
Search near sinks for:
- `shell=False`
- fixed command list
- `args = [`
- `allowed_commands`
- `allowed_tools`
- `Enum`
- `Literal`
- `choices=`
- `validate`
- `re.fullmatch`
- `shlex.quote` with context review
- `timeout=`
- `cwd=`
- safe working directory
- environment allowlist
- `env={`
- `resource.setrlimit`
- `subprocess.DEVNULL`
- restricted globals
- `{"__builtins__": {}}`
- `ast.literal_eval`
- sandbox
- non-root

## 2.8 Python graph search recipes
Useful combinations:

```text
@app.route + subprocess.run
request.args + os.system
FastAPI + shell=True
APIView + eval
@shared_task + subprocess.Popen
BaseCommand.handle + os.popen
tool_name + subprocess.run
shell=True + f-string
asyncio.create_subprocess_shell + request
render_template_string + request
ffmpeg + user-controlled option
```

---

# 3. Python Command Execution Anti-Patterns

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

# 4. Case Templates

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

# 5. Python-Specific Audit Heuristics

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
