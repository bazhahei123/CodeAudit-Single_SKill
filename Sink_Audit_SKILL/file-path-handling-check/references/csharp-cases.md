# C# and .NET Path Traversal Cases

## Purpose

This file contains C#/.NET-specific path traversal, arbitrary file access, unsafe file path handling, archive extraction traversal, and local resource loading cases.

Use it when the target application is primarily implemented in C# or .NET, especially in:
- ASP.NET MVC, Web API, Razor Pages, minimal APIs, and ASP.NET Core
- SignalR hubs, gRPC services, WCF services, Azure Functions, and background workers
- upload/download, import/export, report/log/config/template, and file-management features
- ZIP/archive extraction and local resource loading

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real path containment behavior in the target code.

---

# 1. C# / .NET Path Control Points

## 1.1 Entry and restore points
Look for:
- ASP.NET controllers and minimal API endpoints
- Razor Page handlers
- SignalR hub methods
- gRPC/WCF service methods
- Azure Functions triggers
- queue consumers and background workers
- uploaded files and import/export jobs

## 1.2 File operation points
Look for:
- read/download/preview paths
- upload-save and export destination logic
- delete/move/rename/copy handlers
- archive extraction helpers
- template/resource/config/log loading

## 1.3 Path construction and validation
Look for:
- `Path.Combine`
- `Path.GetFullPath`
- `Path.Join`
- `IWebHostEnvironment`
- storage helper wrappers
- canonical containment checks
- symlink/reparse point handling

---

# 2. High-Coverage C# / .NET Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, path construction behavior, file sink behavior, and missing containment controls.

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
- `IFormFileCollection`
- `MapGet`
- `MapPost`
- `MapPut`
- `MapPatch`
- `MapDelete`
- `MapMethods`
- `PageModel`
- `OnGet`
- `OnPost`
- `OnPostAsync`

## 2.2 RPC, function, worker, queue, and admin entries
Search for:
- `Hub`
- `Hub<T>`
- `OnConnectedAsync`
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
- `Import`
- `Export`
- `Upload`
- `Download`
- `Preview`
- `Delete`
- `Cleanup`

## 2.3 Path construction and transformation candidates
Search for:
- `Path.Combine`
- `Path.Join`
- `Path.GetFullPath`
- `Path.GetRelativePath`
- `Path.GetFileName`
- `Path.GetDirectoryName`
- `Path.GetExtension`
- `Path.ChangeExtension`
- `Path.DirectorySeparatorChar`
- `Path.AltDirectorySeparatorChar`
- `Uri.UnescapeDataString`
- `WebUtility.UrlDecode`
- `HttpUtility.UrlDecode`
- `Convert.FromBase64String`
- `IWebHostEnvironment.WebRootPath`
- `IWebHostEnvironment.ContentRootPath`
- `Server.MapPath`
- `MapPath`
- `FileName`
- `ContentDispositionHeaderValue`
- `IFormFile.FileName`
- `Replace(\"../\"`
- `Replace(\"..\"`
- `StringBuilder`
- `$\"{`

## 2.4 File read, preview, download, and resource-load sink candidates
Search for:
- `System.IO.File.ReadAllBytes`
- `System.IO.File.ReadAllText`
- `System.IO.File.ReadLines`
- `System.IO.File.OpenRead`
- `System.IO.File.Open`
- `FileStream`
- `StreamReader`
- `PhysicalFile`
- `File`
- `FileResult`
- `PhysicalFileResult`
- `FileStreamResult`
- `VirtualFileResult`
- `Results.File`
- `SendFileAsync`
- `IFileProvider.GetFileInfo`
- `PhysicalFileProvider`
- `StaticFileMiddleware`
- `Image.FromFile`
- `XDocument.Load`
- `ConfigurationBuilder.AddJsonFile`
- `RazorViewEngine`

## 2.5 File write, upload-save, copy, move, delete, and metadata sink candidates
Search for:
- `System.IO.File.WriteAllBytes`
- `System.IO.File.WriteAllText`
- `System.IO.File.AppendAllText`
- `System.IO.File.Create`
- `System.IO.File.Delete`
- `System.IO.File.Copy`
- `System.IO.File.Move`
- `Directory.CreateDirectory`
- `Directory.Delete`
- `Directory.Move`
- `FileInfo.Delete`
- `FileInfo.MoveTo`
- `FileInfo.CopyTo`
- `DirectoryInfo.Delete`
- `IFormFile.CopyTo`
- `IFormFile.CopyToAsync`
- `Path.GetTempFileName`
- `Path.GetRandomFileName`
- `File.SetAttributes`
- `File.Exists`
- `Directory.Exists`

## 2.6 Archive extraction candidates
Search for:
- `ZipFile.ExtractToDirectory`
- `ZipArchive`
- `ZipArchiveEntry`
- `entry.FullName`
- `entry.Name`
- `ExtractToFile`
- `OpenRead`
- `SharpCompress`
- `TarFile.ExtractToDirectory`
- `GZipStream`
- `DeflateStream`
- `extract`
- `unzip`

## 2.7 Required-control candidates
Search near sinks for:
- `Path.GetFullPath`
- `StartsWith`
- `StringComparison.Ordinal`
- `Path.GetFileName`
- `GetRelativePath`
- `ContentRootPath`
- `WebRootPath`
- `basePath`
- `rootPath`
- `allowedExtensions`
- `allowedContentTypes`
- `IFileProvider`
- `LinkTarget`
- `FileAttributes.ReparsePoint`
- `EnumerationOptions`
- `IgnoreInaccessible`
- `validate`
- `allowlist`
- `Guid.NewGuid`
- `Path.GetRandomFileName`
- `RequestSizeLimit`

## 2.8 C# / .NET graph search recipes
Useful combinations:

```text
[HttpGet] + PhysicalFile
[HttpGet] + File.ReadAllBytes
[FromQuery] + Path.Combine
[HttpPost] + IFormFile.CopyToAsync
IFormFile.FileName + Path.Combine
MapDelete + File.Delete
QueueTrigger + File.WriteAllText
BlobTrigger + Path.Combine
ZipFile.ExtractToDirectory
ZipArchiveEntry.FullName + ExtractToFile
Path.GetFullPath + StartsWith
```

---

# 3. C# / .NET Path Traversal Anti-Patterns

### A1. Joined download path from request
```csharp
var path = Path.Combine(baseDir, fileName);
return PhysicalFile(path, "application/octet-stream");
```

Why risky:
User-controlled `fileName` may escape the intended base directory without resolved containment.

### A2. Uploaded filename used as destination
```csharp
var path = Path.Combine(uploadDir, formFile.FileName);
await formFile.CopyToAsync(new FileStream(path, FileMode.Create));
```

Why risky:
Client-supplied upload filenames can contain traversal or unsafe path semantics.

### A3. Unsafe delete path
```csharp
System.IO.File.Delete(Path.Combine(baseDir, name));
```

Why risky:
Weak containment allows attacker-controlled deletion outside the intended scope.

### A4. Archive extraction without entry validation
```csharp
ZipFile.ExtractToDirectory(zipPath, destDir);
```

Why risky:
Archive entries may overwrite or escape intended extraction roots if not validated.

---

# 4. Case Templates

## Case C-PATH-1: Arbitrary read via PhysicalFile

### Vulnerable pattern
```csharp
return PhysicalFile(Path.Combine(baseDir, file), "application/octet-stream");
```

### Audit focus
Verify request control, full-path containment, symlink/reparse point behavior, and file disclosure impact.

## Case C-PATH-2: Upload overwrite via client filename

### Vulnerable pattern
```csharp
var path = Path.Combine(uploadDir, file.FileName);
```

### Audit focus
Verify server-generated filename use, extension allowlists, overwrite behavior, and base containment.

## Case C-PATH-3: Unsafe archive extraction

### Vulnerable pattern
```csharp
entry.ExtractToFile(Path.Combine(destDir, entry.FullName));
```

### Audit focus
Verify each entry path is resolved under the extraction root before writing.

---

# 5. C# / .NET-Specific Audit Heuristics

## 5.1 Path API heuristics
Pay attention to `Path.Combine`, `Path.GetFullPath`, prefix checks, alternate separators, and reparse points.

## 5.2 File result heuristics
Review `PhysicalFile`, `FileResult`, `Results.File`, static file providers, and download helpers.

## 5.3 Upload heuristics
Treat `IFormFile.FileName` as attacker-controlled and prefer generated names.

## 5.4 Archive heuristics
Review `ZipArchiveEntry.FullName`, `ExtractToFile`, and whole-directory extraction helpers.
