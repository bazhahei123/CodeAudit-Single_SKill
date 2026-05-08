# C# and .NET Command Execution Source Cases

## Purpose

This file contains C#/.NET-specific source point patterns and candidate search terms for command execution and code execution source discovery.

Use it when the target application is primarily implemented in C# or .NET, especially:
- ASP.NET Core MVC / Web API
- Razor Pages and minimal APIs
- SignalR hubs
- gRPC and WCF services
- Azure Functions and queue consumers
- hosted services, Hangfire, MassTransit, NServiceBus, MediatR handlers
- process wrappers, PowerShell runners, Roslyn scripting, expression evaluators, external tool wrappers, and automation jobs

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. High-Coverage C#/.NET Source Candidate Inventory

## 1.1 HTTP, API, RPC, and worker entry candidates

- `[ApiController]`
- `[Route]`
- `[HttpGet]`
- `[HttpPost]`
- `[HttpPut]`
- `[HttpPatch]`
- `[HttpDelete]`
- Razor Pages `PageModel`
- `OnGet`
- `OnPost`
- `MapGet`
- `MapPost`
- `MapPut`
- `MapPatch`
- `MapDelete`
- SignalR `Hub`
- hub method arguments
- gRPC service methods
- WCF `[ServiceContract]`
- WCF `[OperationContract]`
- Azure Functions `[HttpTrigger]`
- Azure Functions `[QueueTrigger]`
- Azure Functions `[ServiceBusTrigger]`
- Azure Functions `[TimerTrigger]`
- hosted services
- Hangfire jobs
- MassTransit consumers
- NServiceBus handlers
- MediatR handlers

## 1.2 Request, queue, and payload source candidates

- `[FromRoute]`
- `[FromQuery]`
- `[FromBody]`
- `[FromForm]`
- `[FromHeader]`
- `HttpRequest.Query`
- `HttpRequest.Form`
- `HttpRequest.Headers`
- `HttpRequest.Cookies`
- `RouteData.Values`
- `IFormFile.FileName`
- DTO fields
- SignalR method args
- protobuf request fields
- queue message bodies
- job args
- function trigger payloads

## 1.3 Command, tool, script, and action selector candidates

- `cmd`
- `command`
- `commandName`
- `tool`
- `toolName`
- `executable`
- `program`
- `fileName`
- `process`
- `script`
- `scriptName`
- `scriptPath`
- `interpreter`
- `runtime`
- `engine`
- `subcommand`
- `action`
- `operation`
- `mode`
- `runner`
- `plugin`
- `taskName`
- `jobType`

## 1.4 Argument, option, path, environment, and cwd candidates

- `arg`
- `args`
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
- `workingDirectory`
- `cwd`
- `stdin`
- `payload`
- `timeout`
- `profile`

## 1.5 Eval, script, PowerShell, and expression candidates

- `commandLine`
- `commandTemplate`
- `scriptBody`
- `sourceCode`
- `code`
- `expression`
- `formula`
- `rule`
- `PowerShell`
- `AddScript`
- `AddCommand`
- `CSharpScript`
- `Roslyn`
- `DynamicExpressionParser`
- `DataTable.Compute`
- `NCalc`
- `Jint`
- `ClearScript`
- `IronPython`
- `RazorEngine`

## 1.6 Downstream execution mapping candidates

- `Process.Start`
- `ProcessStartInfo`
- `FileName`
- `Arguments`
- `ArgumentList`
- `UseShellExecute`
- `WorkingDirectory`
- `EnvironmentVariables`
- `RedirectStandardInput`
- `cmd.exe`
- `powershell`
- `pwsh`
- `bash`
- `PowerShell.Create`
- `CSharpScript.EvaluateAsync`
- `CSharpScript.RunAsync`
- `Assembly.Load`
- `Activator.CreateInstance`
- external tool wrappers
- `ffmpeg`
- `magick`
- `pandoc`
- `wkhtmltopdf`

## 1.7 C#/.NET graph search recipes

```text
[HttpPost]/MapPost + FromBody cmd/tool/action + Process.Start/ProcessStartInfo
FromQuery/FromBody + args/options/file/path + ArgumentList/Arguments
commandTemplate/string interpolation + cmd.exe/powershell/UseShellExecute
PowerShell/Roslyn/NCalc/Jint + expression/script/code + request/stored value
QueueTrigger/ServiceBusTrigger/Hangfire + job args/commandTemplate + process wrapper
ProcessStartInfo.EnvironmentVariables/WorkingDirectory + env/cwd/config/path + request/queue/stored source
```

---

# 2. C#/.NET Source Patterns and Blind Spots

## C-S1. Request command and argument values

Route, query, body, form, header, SignalR, gRPC, and function payload values become source points when they flow into `ProcessStartInfo`, `Process.Start`, PowerShell, scripts, or tool wrappers.

## C-S2. PowerShell and Roslyn sources

PowerShell script text, C# script bodies, expressions, and dynamic assembly/class names are high-priority code-execution source candidates.

## C-S3. Queue and job source

Function triggers, Hangfire jobs, queue messages, and service bus payloads can be second-order execution sources.

## C-S4. Environment and working directory source

Environment variables, working directories, stdin, config files, and tool paths can alter execution behavior even when executable names are fixed.

---

# 3. False-Positive Controls

Do not mark a C#/.NET source as high-priority if:
- the command/tool/action is selected from a strict allowlist,
- the source only affects non-execution display/logging,
- script/expression content is fixed trusted code,
- downstream execution relevance is absent.

Use `Suspected source` or `Not enough evidence` if:
- wrapper helper behavior is hidden,
- PowerShell/Roslyn/expression restrictions are hidden,
- queue/job writer paths are missing,
- `UseShellExecute`, shell context, or argument handling is unclear.

---

# 4. Quick C#/.NET Source Checklist

- Are request, SignalR, gRPC, queue, or function values used as command names, arguments, options, paths, or tool selectors?
- Are `ProcessStartInfo.FileName`, `Arguments`, `ArgumentList`, environment, or working directory influenced?
- Are PowerShell, Roslyn, dynamic expressions, dynamic assemblies, or script engines fed by external or stored input?
- Are stored jobs, automation rules, or replay payloads later executed?
- Are wrappers hiding the real execution boundary?
