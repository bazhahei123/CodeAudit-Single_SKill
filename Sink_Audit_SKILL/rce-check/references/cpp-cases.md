# C++ Command Execution Cases

## Purpose

This file contains C++-specific command execution, shell invocation, dynamic library loading, plugin loading, interpreter embedding, and external-tool execution cases.

Use it when the target application includes C++ or native service code, especially in:
- HTTP, RPC, WebSocket, IPC, socket, or message-bus handlers
- diagnostic, admin, conversion, import/export, report, media, or automation features
- process spawning through C, POSIX, Windows, Qt, Boost, Poco, or custom wrappers
- dynamic library, plugin, script, or interpreter loading paths

This reference is guidance, not proof. C++ execution issues often combine command injection with privilege, environment, path, or memory-safety boundaries. Always verify attacker influence, execution context, sink behavior, and missing controls.

---

# 1. C++ Execution Control Points

## 1.1 Entry and task points
Look for:
- HTTP route handlers
- RPC or gRPC service methods
- socket, pipe, DBus, Binder, or custom IPC handlers
- message queue consumers
- admin, diagnostics, import/export, and conversion paths
- external-tool wrappers
- plugin, script, and dynamic library loading

## 1.2 Execution points
Look for:
- C stdlib process APIs
- POSIX process APIs
- Windows process APIs
- Qt and Boost process wrappers
- shell wrappers
- embedded interpreters
- dynamic loading APIs

## 1.3 Trust-boundary controls
Look for:
- fixed executable paths
- argv arrays and no shell
- option/tool allowlists
- controlled environment and working directory
- sandboxing, privilege drop, and timeouts
- signed or trusted plugin/library loading

---

# 2. High-Coverage C++ Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, execution context, sink behavior, and missing controls.

## 2.1 HTTP, RPC, WebSocket, and IPC entry candidates
Search for:
- `CROW_ROUTE`
- `crow::SimpleApp`
- `app.route_dynamic`
- `DROGON_BEGIN_NAMESPACE`
- `METHOD_LIST_BEGIN`
- `ADD_METHOD_TO`
- `HttpController`
- `HttpSimpleController`
- `HttpRequestPtr`
- `HttpResponsePtr`
- `oatpp::web::server::api::Controller`
- `ENDPOINT`
- `Pistache::Rest::Routes::Post`
- `Pistache::Rest::Routes::Get`
- `httplib::Server`
- `svr.Post`
- `svr.Get`
- `boost::beast`
- `http_listener`
- `grpc::Service`
- `ServerContext`
- `apache::thrift`
- `TProcessor`
- `onMessage`
- `websocket`
- `DBus`
- `sd_bus_message`
- `Binder`
- `onTransact`
- `recv`
- `recvfrom`
- `read`

## 2.2 Message, import/export, conversion, and admin entries
Search for:
- `KafkaConsumer`
- `RdKafka`
- `AMQP`
- `RabbitMQ`
- `MQTT`
- `ZeroMQ`
- `zmq_recv`
- `boost::asio`
- `admin`
- `diagnostic`
- `convert`
- `render`
- `import`
- `export`
- `scan`
- `preview`
- `generate`
- `backup`
- `restore`
- `replay`
- `worker`
- `job`

## 2.3 Process, shell, POSIX, Windows, and framework sink candidates
Search for:
- `system`
- `popen`
- `_popen`
- `std::system`
- `execl`
- `execlp`
- `execle`
- `execv`
- `execvp`
- `execve`
- `posix_spawn`
- `fork`
- `vfork`
- `CreateProcess`
- `ShellExecute`
- `ShellExecuteEx`
- `WinExec`
- `QProcess`
- `QProcess::start`
- `QProcess::execute`
- `QProcess::startDetached`
- `boost::process`
- `bp::child`
- `Poco::Process::launch`
- `sh -c`
- `bash -c`
- `cmd.exe`
- `powershell`

## 2.4 Dynamic code, plugin, interpreter, and library loading candidates
Search for:
- `dlopen`
- `dlsym`
- `LoadLibrary`
- `GetProcAddress`
- `QPluginLoader`
- `QLibrary`
- `std::filesystem` plugin path review
- `luaL_dostring`
- `luaL_loadstring`
- `lua_pcall`
- `PyRun_SimpleString`
- `PyRun_String`
- `PyImport_ImportModule`
- `JS_EvaluateScript`
- `v8::Script::Compile`
- `Chakra`
- `AngelScript`
- `Tcl_Eval`
- `system` wrappers
- `runCommand`
- `executeCommand`
- `invokeTool`

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
- `toolName`
- `executable`
- `script`
- `scriptPath`
- `commandTemplate`
- `std::string`
- `std::stringstream`
- `fmt::format`
- `absl::StrFormat`
- `QString`
- `QString::arg`
- `join`
- `split`
- `strcat`
- `sprintf`
- `snprintf`
- `URLDecode`
- `QUrl::fromPercentEncoding`
- `base64_decode`
- `workingDirectory`
- `environment`
- `envp`

## 2.7 Required-control candidates
Search near sinks for:
- fixed executable path
- argv array
- no `sh -c`
- no `cmd.exe`
- command allowlist
- tool allowlist
- option allowlist
- `enum`
- `switch`
- `regex_match`
- `std::regex_match`
- `QRegularExpression`
- `posix_spawn` argv review
- `CreateProcess` quoted arguments review
- clean environment
- working directory allowlist
- timeout
- kill process
- `setrlimit`
- `seccomp`
- `chroot`
- `drop privileges`
- signature verification

## 2.8 C++ graph search recipes
Useful combinations:

```text
CROW_ROUTE + system
svr.Post + popen
grpc::Service + CreateProcess
recv + QProcess::start
admin + boost::process
diagnostic + ping + system
QProcess + sh -c
std::string command + system
dlopen + request-controlled path
luaL_dostring + user script
toolName + Poco::Process::launch
ffmpeg + user-controlled option
```

---

# 3. C++ Command Execution Anti-Patterns

### A1. HTTP parameter reaches system
```cpp
std::string cmd = "ping " + host;
system(cmd.c_str());
```

Why risky:
User-controlled input reaches a shell command string.

### A2. Dynamic executable selection
```cpp
boost::process::child c(toolName, arg);
```

Why risky:
Attacker influence over executable selection controls what runs.

### A3. QProcess starts shell with user content
```cpp
QProcess::startDetached("sh", QStringList() << "-c" << script);
```

Why risky:
Shell interpretation enables separators, substitutions, expansions, and quoting bypasses.

### A4. Dynamic library loading from untrusted path
```cpp
void* h = dlopen(path.c_str(), RTLD_NOW);
```

Why risky:
Loading attacker-controlled native libraries is direct code execution.

---

# 4. Case Templates

## Case CPP-CMD-1: Route command injection

### Vulnerable pattern
```cpp
system(("ping " + req.get_param_value("host")).c_str());
```

### Audit focus
Verify external reachability, shell context, command construction, and allowlist controls.

## Case CPP-CMD-2: Dynamic process wrapper

### Vulnerable pattern
```cpp
Poco::Process::launch(toolName, args);
```

### Audit focus
Verify fixed executable selection, option allowlists, environment, and working directory.

## Case CPP-CMD-3: Dynamic plugin or script loading

### Vulnerable pattern
```cpp
QLibrary lib(userPath);
lib.load();
```

### Audit focus
Verify path trust, signature validation, plugin allowlists, and attacker influence.

---

# 5. C++-Specific Audit Heuristics

## 5.1 Process API heuristics
Review `system`, `popen`, `exec*`, `posix_spawn`, `CreateProcess`, `ShellExecute`, `QProcess`, Boost.Process, and Poco.Process.

## 5.2 Shell heuristics
Treat `sh -c`, `bash -c`, `cmd.exe`, and PowerShell invocations as high-risk when input affects the command string.

## 5.3 Dynamic loading heuristics
Review `dlopen`, `LoadLibrary`, `QPluginLoader`, embedded interpreters, and plugin registries.

## 5.4 External tool heuristics
Review conversion, diagnostics, scan, media, archive, and admin wrappers for command, option, path, environment, and working-directory control.
