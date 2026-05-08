# C# and .NET Unsafe Deserialization Source Cases

## Purpose

This file contains C#/.NET-specific source point patterns and candidate search terms for unsafe deserialization source discovery.

Use it when the target application is primarily implemented in C# or .NET, especially:
- ASP.NET Core MVC / Web API
- Razor Pages and minimal APIs
- SignalR hubs
- gRPC and WCF services
- Azure Functions and queue consumers
- cache/session restore helpers
- message consumers and background workers
- .NET code that receives JSON/XML/YAML/binary payloads, `$type` metadata, or object blobs before restoration

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
- `OnPost`
- `MapPost`
- `MapPut`
- `MapPatch`
- `MapMethods`
- SignalR `Hub`
- hub method arguments
- gRPC service methods
- WCF `[ServiceContract]`
- WCF `[OperationContract]`
- Azure Functions `[HttpTrigger]`
- Azure Functions `[QueueTrigger]`
- Azure Functions `[ServiceBusTrigger]`
- Azure Functions `[EventGridTrigger]`
- MassTransit consumers
- NServiceBus handlers
- MediatR handlers
- hosted services
- Hangfire jobs

## 1.2 Request and payload source candidates

- `[FromBody]`
- `[FromForm]`
- `[FromRoute]`
- `[FromQuery]`
- `[FromHeader]`
- `HttpRequest.Body`
- `Request.Form`
- `Request.Headers`
- `Request.Cookies`
- `IFormFile`
- `Stream`
- `byte[]`
- `ReadOnlyMemory<byte>`
- `BinaryData`
- `JsonElement`
- `JObject`
- `XElement`
- queue message bodies
- protobuf request fields
- SignalR method arguments

## 1.3 Serialized payload and wrapper candidates

- `payload`
- `data`
- `body`
- `message`
- `blob`
- `bytes`
- `raw`
- `content`
- `serialized`
- `serializedData`
- `objectData`
- `binary`
- `base64`
- `gzip`
- `zip`
- `compressed`
- `encoded`
- `encrypted`
- `signed`
- `cookie`
- `session`
- `token`
- `metadata`
- `template`
- `config`
- `backup`
- `snapshot`

## 1.4 Type and polymorphic metadata candidates

- `$type`
- `TypeNameHandling`
- `SerializationBinder`
- `ISerializationBinder`
- `KnownTypes`
- `DataContract`
- `DataMember`
- `NetDataContractSerializer`
- `BinaryFormatter`
- `LosFormatter`
- `ObjectStateFormatter`
- `SoapFormatter`
- `DataSet`
- `DataTable`
- `JObject.ToObject`
- `JsonSerializer.Deserialize<object>`
- `object`
- `dynamic`
- `type`
- `class`
- `assembly`
- `AssemblyQualifiedName`
- `targetType`
- `discriminator`

## 1.5 Stored, cache, session, and second-order candidates

- ASP.NET session values
- cookies
- distributed cache
- `IDistributedCache`
- Redis values
- database blob fields
- EF Core `byte[]` columns
- JSON metadata fields
- saved filters
- saved templates
- saved configs
- uploaded archives
- queue messages
- dead-letter messages
- retry payloads
- Hangfire job args
- Azure Function replay/admin payloads
- object storage blobs

## 1.6 Restore and downstream mapping candidates

- `BinaryFormatter.Deserialize`
- `NetDataContractSerializer.ReadObject`
- `DataContractSerializer.ReadObject`
- `XmlSerializer.Deserialize`
- `SoapFormatter.Deserialize`
- `LosFormatter.Deserialize`
- `ObjectStateFormatter.Deserialize`
- `JsonConvert.DeserializeObject`
- `JsonSerializer.Deserialize`
- `JObject.ToObject`
- `YamlDotNet`
- `MessagePackSerializer.Deserialize`
- `protobuf-net Serializer.Deserialize`
- `MemoryStream`
- `GZipStream`
- `Convert.FromBase64String`
- custom `Deserialize`
- custom `Restore`
- custom `Hydrate`

## 1.7 C#/.NET graph search recipes

```text
[HttpPost]/MapPost + FromBody payload/data/blob + Convert.FromBase64String/GZipStream
Request.Cookies/Session + serialized/session/token + Deserialize/ReadObject
JsonConvert.DeserializeObject + TypeNameHandling/$type/object/dynamic + request body
QueueTrigger/ServiceBusTrigger + message/body + BinaryFormatter/JsonConvert/YamlDotNet
IDistributedCache/Redis/database blob + value + Deserialize/Restore
IFormFile/import/config + stream + XmlSerializer/NetDataContractSerializer/YamlDotNet
```

---

# 2. C#/.NET Source Patterns and Blind Spots

## C-S1. Request body or file stream source

HTTP bodies and file streams become source points when they flow into serializers, object mappers, XML/YAML loaders, or binary decoders.

## C-S2. `$type` and TypeNameHandling source

Newtonsoft `$type`, assembly-qualified names, `object` targets, and `dynamic` mappings are high-priority type-restoration source candidates.

## C-S3. Queue, function, and SignalR source

Queue messages, Azure Function payloads, and SignalR hub arguments are weakly trusted until producer and caller constraints are proven.

## C-S4. Cache/session stored blob source

Distributed cache, session, cookie, Redis, and database blobs can be second-order source material.

---

# 3. False-Positive Controls

Do not mark a C#/.NET source as high-priority if:
- the value is only parsed into strict primitive DTOs,
- polymorphic type handling is disabled or safely bound and visible,
- stored values are trusted-only and integrity protected,
- downstream deserialization relevance is absent.

Use `Suspected source` or `Not enough evidence` if:
- JSON serializer settings are hidden,
- cache/session writer paths are missing,
- queue producer trust is unclear,
- helper wrappers hide the real serializer.

---

# 4. Quick C#/.NET Source Checklist

- Are bodies, uploads, cookies, sessions, queue messages, or SignalR/gRPC arguments deserialized into object-like targets?
- Are `$type`, assembly names, `object`, `dynamic`, or polymorphic discriminator fields accepted?
- Are cache, Redis, session, or database blobs restored later?
- Are Azure Functions or workers treating provider messages as trusted?
- Are restored object fields path, URL, command, callback, type, method, or template values?
