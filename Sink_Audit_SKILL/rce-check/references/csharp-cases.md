# C# and .NET Command Execution Cases

## Purpose

This file contains C#/.NET-specific command execution, shell invocation, interpreter abuse, expression evaluation, dynamic assembly loading, and external-tool execution cases.

Use it when the target application is primarily implemented in C# or .NET, especially in:
- ASP.NET MVC, Web API, Razor Pages, minimal APIs, and ASP.NET Core
- SignalR hubs, gRPC services, WCF services, Azure Functions, and background workers
- admin, diagnostics, import/export, report generation, conversion, and automation features
- Roslyn scripting, PowerShell, dynamic compilation, reflection, plugin loading, and process wrappers

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real execution boundary in the target code.

---

# 1. C# / .NET Execution Control Points

## 1.1 Entry and process points
Look for:
- ASP.NET controllers and minimal API endpoints
- Razor Page handlers
- SignalR hub methods
- gRPC/WCF service methods
- Azure Functions triggers
- queue consumers and background workers
- admin and diagnostic handlers
- report/conversion/automation wrappers

## 1.2 Execution points
Look for:
- process launch APIs
- shell wrappers
- PowerShell execution
- Roslyn scripting and dynamic compilation
- reflection and assembly loading
- template/expression engines
- external tool wrappers

## 1.3 Trust-boundary controls
Look for:
- fixed executable names
- array-style arguments
- `UseShellExecute = false`
- tool/option allowlists
- restricted PowerShell runspaces
- trusted-only assembly/plugin loading
- timeouts and resource limits

---

# 2. High-Coverage C# / .NET Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, execution context, sink behavior, and missing controls.

## 2.1 ASP.NET, Web API, Razor Pages, and minimal API entry candidates
Search for:
- `[ApiController]`
- `[Controller]`
- `[Route]`
- `[HttpGet]`
- `[HttpPost]`
- `[HttpPut]`
- `[HttpPatch]`
- `[HttpDelete]`
- `[FromBody]`
- `[FromForm]`
- `[FromQuery]`
- `[FromHeader]`
- `[FromRoute]`
- `ControllerBase`
- `Controller`
- `IActionResult`
- `ActionResult`
- `HttpRequest`
- `Request.Query`
- `Request.Form`
- `Request.Cookies`
- `Request.Headers`
- `IFormFile`
- `MapGet`
- `MapPost`
- `MapPut`
- `MapPatch`
- `MapDelete`
- `PageModel`
- `OnGet`
- `OnPost`

## 2.2 RPC, function, worker, queue, and admin entries
Search for:
- `Hub`
- `Hub<T>`
- `GrpcService`
- `ServerCallContext`
- `[OperationContract]`
- `[ServiceContract]`
- `[WebMethod]`
- `IHttpHandler`
- `ProcessRequest`
- `HttpTrigger`
- `QueueTrigger`
- `ServiceBusTrigger`
- `BlobTrigger`
- `TimerTrigger`
- `IHostedService`
- `BackgroundService`
- `ExecuteAsync`
- `Hangfire`
- `RecurringJob`
- `MassTransit`
- `IConsumer`
- `Handle`
- `Consume`
- `Admin`
- `Diagnostic`
- `Convert`
- `Render`
- `Import`
- `Export`
- `Replay`

## 2.3 Process, shell, and OS command sink candidates
Search for:
- `System.Diagnostics.Process.Start`
- `Process.Start`
- `ProcessStartInfo`
- `ProcessStartInfo.FileName`
- `ProcessStartInfo.Arguments`
- `ArgumentList`
- `UseShellExecute`
- `RedirectStandardOutput`
- `cmd.exe`
- `/c`
- `powershell.exe`
- `pwsh`
- `PowerShell.Create`
- `AddScript`
- `AddCommand`
- `Invoke`
- `Runspace`
- `RunspaceFactory`
- `bash`
- `sh`
- `WSL`
- `ShellExecute`
- `runCommand`
- `executeCommand`
- `invokeTool`

## 2.4 Eval, scripting, expression, reflection, and dynamic loading candidates
Search for:
- `CSharpScript.EvaluateAsync`
- `CSharpScript.RunAsync`
- `ScriptOptions`
- `Microsoft.CodeAnalysis.CSharp.Scripting`
- `CodeDomProvider`
- `CSharpCodeProvider`
- `CompileAssemblyFromSource`
- `Assembly.Load`
- `Assembly.LoadFrom`
- `AssemblyLoadContext`
- `Activator.CreateInstance`
- `Type.GetType`
- `MethodInfo.Invoke`
- `PropertyInfo.SetValue`
- `Expression.Compile`
- `DataTable.Compute`
- `DynamicExpressionParser`
- `System.Linq.Dynamic`
- `Jint`
- `Jurassic`
- `ClearScript`
- `XslCompiledTransform`
- `XsltSettings.EnableScript`
- `RazorLight`
- `Scriban`

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
- `7z`
- `git`
- `ssh`
- `scp`
- `curl`
- `wget`
- `ping`
- `tracert`
- `nslookup`
- `openssl`
- `docker`
- `kubectl`

## 2.6 Command and argument construction candidates
Search for:
- `cmd`
- `command`
- `arguments`
- `args`
- `toolName`
- `executable`
- `script`
- `scriptPath`
- `commandTemplate`
- `StringBuilder`
- `string.Format`
- `$"{`
- `string.Join`
- `Split(' ')`
- `Uri.UnescapeDataString`
- `WebUtility.UrlDecode`
- `Convert.FromBase64String`
- `WorkingDirectory`
- `EnvironmentVariables`
- `FileName`
- `Arguments`
- `ArgumentList.Add`
- `allowedCommands`
- `allowedTools`

## 2.7 Required-control candidates
Search near sinks for:
- `UseShellExecute = false`
- `ArgumentList`
- fixed `FileName`
- no `cmd.exe`
- no `powershell.exe`
- command allowlist
- tool allowlist
- option allowlist
- `Enum`
- `switch`
- `Regex.IsMatch`
- `Validate`
- `CancellationToken`
- `WaitForExit`
- `Kill`
- timeout
- restricted `Runspace`
- `InitialSessionState`
- `ScriptOptions` restrictions
- environment allowlist
- working directory allowlist
- non-admin identity

## 2.8 C# / .NET graph search recipes
Useful combinations:

```text
[HttpPost] + Process.Start
[FromQuery] + ProcessStartInfo.Arguments
MapGet + cmd.exe + /c
HttpTrigger + powershell
QueueTrigger + Process.Start
BackgroundService + shell command
PowerShell.Create + AddScript
CSharpScript.EvaluateAsync + request
Assembly.LoadFrom + uploaded path
toolName + ProcessStartInfo.FileName
```

---

# 3. C# / .NET Command Execution Anti-Patterns

### A1. Process.Start with concatenated arguments
```csharp
Process.Start("cmd.exe", "/c ping " + host);
```

Why risky:
User-controlled input reaches a shell context.

### A2. Dynamic executable selection
```csharp
Process.Start(new ProcessStartInfo { FileName = toolName, Arguments = args });
```

Why risky:
Attacker influence over the executable changes what program runs.

### A3. PowerShell AddScript with user input
```csharp
PowerShell.Create().AddScript(script).Invoke();
```

Why risky:
Attacker-controlled script content can execute directly.

### A4. Roslyn scripting from request data
```csharp
await CSharpScript.EvaluateAsync(expression);
```

Why risky:
Expression evaluation can become direct code execution without a strict safe subset.

---

# 4. Case Templates

## Case C-CMD-1: ASP.NET route to shell

### Vulnerable pattern
```csharp
Process.Start("cmd.exe", "/c " + command);
```

### Audit focus
Verify request influence, shell context, executable/argument allowlists, and timeout controls.

## Case C-CMD-2: Azure Function to external tool

### Vulnerable pattern
```csharp
Process.Start(toolName, args);
```

### Audit focus
Verify trigger source, command allowlist, option allowlist, and working directory/environment controls.

## Case C-CMD-3: Dynamic script evaluation

### Vulnerable pattern
```csharp
CSharpScript.EvaluateAsync(userExpression);
```

### Audit focus
Verify safe expression constraints, globals, references, imports, and attacker influence.

---

# 5. C# / .NET-Specific Audit Heuristics

## 5.1 Process API heuristics
Pay attention to `Process.Start`, `ProcessStartInfo`, `FileName`, `Arguments`, `ArgumentList`, and `UseShellExecute`.

## 5.2 PowerShell heuristics
Review `AddScript`, `AddCommand`, runspace restrictions, imported modules, and execution policy assumptions.

## 5.3 Dynamic code heuristics
Review Roslyn scripting, dynamic compilation, reflection, assembly loading, and template engines.

## 5.4 Worker and tool heuristics
Treat background workers, function triggers, queue consumers, report generation, and conversion helpers as first-class execution surfaces.
