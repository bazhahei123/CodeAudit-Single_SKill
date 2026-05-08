# C# and .NET File Path Handling Source Cases

## Purpose

This file contains C#/.NET-specific source point patterns and candidate search terms for path traversal and unsafe file path handling source discovery.

Use it when the target application is primarily implemented in C# or .NET, especially:
- ASP.NET Core MVC / Web API
- Razor Pages and minimal APIs
- SignalR hubs
- gRPC and WCF services
- Azure Functions and queue consumers
- file download/upload/import/export workflows
- archive extraction, template loading, storage providers, and cleanup jobs

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
- Azure Functions `[BlobTrigger]`
- Azure Functions `[ServiceBusTrigger]`
- hosted services
- Hangfire jobs
- MassTransit consumers
- MediatR handlers

## 1.2 Request and upload source candidates

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
- `IFormFile.OpenReadStream`
- `ContentDispositionHeaderValue.FileName`
- DTO fields named file/path/template/resource/export/destination
- SignalR method args
- protobuf request fields
- queue message bodies

## 1.3 Path-like field and selector candidates

- `file`
- `fileName`
- `filename`
- `path`
- `filePath`
- `directory`
- `folder`
- `resource`
- `template`
- `view`
- `theme`
- `locale`
- `report`
- `config`
- `log`
- `storageKey`
- `blobName`
- `container`
- `prefix`
- `uri`
- `url`
- `download`
- `preview`
- `export`
- `destination`
- `target`
- `cleanupTarget`

## 1.4 Path construction and transform candidates

- `Path.Combine`
- `Path.Join`
- `Path.GetFullPath`
- `Path.GetFileName`
- `Path.GetExtension`
- `Path.GetDirectoryName`
- `Path.TrimEndingDirectorySeparator`
- `Uri.UnescapeDataString`
- `WebUtility.UrlDecode`
- `DirectoryInfo`
- `FileInfo`
- `IWebHostEnvironment.WebRootPath`
- `IWebHostEnvironment.ContentRootPath`
- `Server.MapPath`
- `PhysicalFileProvider`
- `EmbeddedFileProvider`
- `IFileProvider`
- `TempPath`
- `UploadPath`
- `DownloadPath`
- `ExtractPath`

## 1.5 Archive, stored, queue, and second-order candidates

- `ZipArchive`
- `ZipArchiveEntry.FullName`
- `ZipFile.ExtractToDirectory`
- `TarFile`
- archive manifests
- EF Core path columns
- file metadata entities
- distributed cache values
- session values
- queue message payloads
- blob metadata
- Azure Blob names
- Hangfire job args
- failed job payloads
- export paths
- cleanup targets
- report paths

## 1.6 Downstream mapping candidates

- `File.ReadAllBytes`
- `File.ReadAllText`
- `File.OpenRead`
- `File.WriteAllBytes`
- `File.WriteAllText`
- `File.Delete`
- `File.Move`
- `File.Copy`
- `Directory.Delete`
- `Directory.CreateDirectory`
- `PhysicalFile`
- `FileStreamResult`
- `Results.File`
- `ControllerBase.File`
- `IFileProvider.GetFileInfo`
- `RazorViewEngine`
- `BlobClient.DownloadTo`
- `BlobClient.Upload`
- `ZipFile.ExtractToDirectory`

## 1.7 C#/.NET graph search recipes

```text
[HttpGet]/MapGet + FromQuery/FromRoute file/path/name + Path.Combine/GetFullPath
IFormFile + FileName + CopyTo/File.Write/BlobClient.Upload
ZipArchiveEntry + FullName + ExtractToDirectory/FileStream
QueueTrigger/ServiceBusTrigger + path/file/exportPath + File.Delete/File.Move
IFileProvider/PhysicalFileProvider + template/resource/view + GetFileInfo/PhysicalFile
EF file metadata/blobName + stored path + Results.File/cleanup/delete
```

---

# 2. C#/.NET Source Patterns and Blind Spots

## C-S1. Request path and filename values

Route, query, body, form, and header values become source points when they flow into `Path.Combine`, `File`, `Directory`, storage, or file response APIs.

## C-S2. Uploaded filename source

`IFormFile.FileName` and content-disposition names are client-controlled metadata unless replaced by generated safe names.

## C-S3. Archive entry source

`ZipArchiveEntry.FullName` and tar entry names are path sources for extraction and overwrite behavior.

## C-S4. Blob/storage key source

Blob names, container names, prefixes, and storage keys are path-like selectors when they map to local files or cloud object operations.

---

# 3. False-Positive Controls

Do not mark a C#/.NET source as high-priority if:
- the value is selected from a strict allowlist of safe resource keys,
- uploaded filenames are replaced by server-generated names,
- values are used only for display/logging,
- downstream path/file operation relevance is absent.

Use `Suspected source` or `Not enough evidence` if:
- shared file helpers hide containment checks,
- storage provider mapping is hidden,
- archive extraction is abstracted,
- queue/blob writer paths are missing.

---

# 4. Quick C#/.NET Source Checklist

- Are route/query/body/header values used as filenames, paths, templates, or storage keys?
- Is `IFormFile.FileName` used for save, export, or later paths?
- Are archive entry names used for extraction destinations?
- Are blob names, EF path fields, cache/session values, or queue messages later used as paths?
- Are delete, move, copy, export, cleanup, and read operations sourced differently?
