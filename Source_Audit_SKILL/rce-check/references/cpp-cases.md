# C++ Command Execution Source Cases

## Purpose

This file contains C++-specific source point patterns and candidate search terms for command execution and code execution source discovery.

Use it when the target application includes C++ code, especially:
- HTTP servers and REST APIs
- CGI/FastCGI handlers
- gRPC / Thrift / custom RPC services
- WebSocket and binary protocol handlers
- native IPC services, plugins, agents, and daemons
- CLI/admin tools, process wrappers, script/plugin runners, expression engines, and external tool integrations

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. High-Coverage C++ Source Candidate Inventory

## 1.1 HTTP, RPC, IPC, and native entry candidates

- cpp-httplib `Get`
- cpp-httplib `Post`
- Crow `CROW_ROUTE`
- Drogon `HttpController`
- Drogon `ADD_METHOD_TO`
- oatpp `ENDPOINT`
- Pistache routes
- Restbed handlers
- Wt resources
- CGI/FastCGI `FCGX_Accept`
- gRPC service methods
- Thrift handlers
- protobuf request messages
- WebSocket message handlers
- DBus methods
- Unix domain socket handlers
- named pipe handlers
- local TCP admin APIs
- plugin callbacks
- queue consumers
- worker jobs
- CLI/import/admin commands

## 1.2 Payload, command, and selector candidates

- route captures
- query parameters
- request body
- headers
- cookies
- protobuf string fields
- IPC message fields
- CLI args
- environment variables in CGI
- `cmd`
- `command`
- `commandName`
- `tool`
- `toolName`
- `program`
- `binary`
- `executable`
- `script`
- `scriptPath`
- `action`
- `operation`
- `mode`
- `plugin`
- `module`
- `handler`

## 1.3 Argument, option, path, environment, and cwd candidates

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
- `workdir`
- `workingDirectory`
- `stdin`
- `payload`
- `timeout`

## 1.4 Shell, script, eval, and plugin candidates

- `commandLine`
- `commandTemplate`
- `shellCommand`
- `scriptBody`
- `sourceCode`
- `expression`
- `formula`
- `rule`
- `lua`
- `python`
- `javascript`
- `pluginPath`
- `libraryPath`
- `moduleName`
- `functionName`
- `symbol`
- `callback`
- `factory`

## 1.5 Downstream execution mapping candidates

- `system`
- `popen`
- `exec`
- `execl`
- `execlp`
- `execle`
- `execv`
- `execvp`
- `posix_spawn`
- `fork`
- `CreateProcess`
- `ShellExecute`
- `QProcess`
- `boost::process`
- `std::system`
- `dlopen`
- `dlsym`
- `LoadLibrary`
- `GetProcAddress`
- Lua execution APIs
- Python C API execution
- JavaScript engine eval
- external tool wrappers

## 1.6 Stored and second-order candidates

- database command fields
- config file commands
- plugin manifests
- workflow definitions
- queue message payloads
- retry payloads
- job args
- saved scripts
- admin replay payloads
- imported project files
- downloaded metadata
- automation profiles
- diagnostic command history

## 1.7 C++ graph search recipes

```text
HTTP/RPC/IPC handler + query/body cmd/tool/action + system/popen/exec/CreateProcess
request/protobuf/IPC + args/options/file/path + QProcess/boost::process
string concat/commandTemplate + shellCommand + system/popen/ShellExecute
pluginPath/module/function + dlopen/dlsym/LoadLibrary/GetProcAddress
queue/job/admin payload + commandTemplate/args + native process wrapper
env/cwd/stdin/config + request/job/stored value + CreateProcess/posix_spawn/QProcess
```

---

# 2. C++ Source Patterns and Blind Spots

## CPP-S1. Request and protocol execution values

HTTP, RPC, IPC, WebSocket, CLI, and plugin callback inputs become source points when they influence commands, arguments, options, scripts, plugins, or external tools.

## CPP-S2. Native process wrapper sources

Values passed to `system`, `popen`, `exec*`, `posix_spawn`, `CreateProcess`, `ShellExecute`, `QProcess`, or `boost::process` need source classification.

## CPP-S3. Plugin and dynamic library sources

Plugin paths, library paths, module names, symbol names, callbacks, and factory keys are execution-relevant when they reach dynamic loading or invocation.

## CPP-S4. Stored automation source

Config files, project files, plugin manifests, queue messages, and saved scripts can be second-order execution sources.

---

# 3. False-Positive Controls

Do not mark a C++ source as high-priority if:
- command/tool/plugin selection is fixed or strictly allowlisted,
- the value only affects display/logging,
- the source cannot be influenced by external or lower-privileged producers,
- downstream execution relevance is absent.

Use `Suspected source` or `Not enough evidence` if:
- native wrapper behavior is hidden,
- plugin/IPC reachability is unclear,
- shell vs argv/process context is unclear,
- stored task writer paths are missing.

---

# 4. Quick C++ Source Checklist

- Are HTTP, RPC, IPC, CLI, plugin, or queue values used as commands, arguments, options, scripts, or tool selectors?
- Are command strings passed to shell-like APIs such as `system`, `popen`, or `ShellExecute`?
- Are dynamic library/plugin paths, module names, symbols, or factory keys influenced?
- Are environment variables, working directories, stdin, config paths, or file paths influenced?
- Are stored jobs, configs, scripts, or replay payloads later executed?
