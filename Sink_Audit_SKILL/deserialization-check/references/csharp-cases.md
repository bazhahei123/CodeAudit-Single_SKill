# C# and .NET Unsafe Deserialization Cases

## Purpose

This file contains C#/.NET-specific unsafe deserialization patterns, candidate sink inventories, and audit cases.

Use it when the target application is primarily implemented in C# or .NET, especially in:
- ASP.NET MVC, Web API, Razor Pages, minimal APIs, and ASP.NET Core
- SignalR hubs, gRPC services, WCF, ASMX, SOAP, and remoting-style services
- Azure Functions, background workers, queue consumers, and scheduled jobs
- desktop/server utilities that import, restore, or replay serialized state
- JSON.NET, XML, binary, SOAP, ViewState, cache, and message serializers

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow, trust boundary, serializer configuration, dangerous trigger behavior, and missing controls.

---

# 1. C# / .NET Deserialization Control Points

## 1.1 Entry and restore points
Look for:
- ASP.NET controllers and minimal API endpoints
- SignalR hub methods
- gRPC, WCF, ASMX, SOAP, and legacy remoting handlers
- Azure Functions triggers
- queue consumers and background workers
- uploaded file, cookie, ViewState, cache, and import restore paths

## 1.2 Object behavior controls
Look for:
- `ISerializable`
- serialization constructors
- `[OnDeserialized]`
- `IDeserializationCallback`
- property setters with side effects
- `SerializationBinder`
- polymorphic type name handling
- post-restore trust in object state

## 1.3 Framework and library controls
Look for:
- BinaryFormatter and formatter wrappers
- JSON.NET TypeNameHandling
- NetDataContractSerializer
- LosFormatter / ObjectStateFormatter
- XAML, XML, SOAP, YAML, MessagePack, and cache serializer configuration
- ViewState MAC and MachineKey handling

## 1.4 Trust-boundary controls
Look for:
- signed vs unsigned cookies
- ViewState integrity
- queue or cache trust assumptions
- imported file trust assumptions
- internal-only service assumptions
- binder or known-type allowlists

---

# 2. High-Coverage C# / .NET Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, sink behavior, trigger behavior, and missing controls.

## 2.1 ASP.NET, Web API, and minimal API entry candidates
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
- `Request.Body`
- `Request.Form`
- `Request.Cookies`
- `Request.Headers`
- `IFormFile`
- `MapGet`
- `MapPost`
- `MapPut`
- `MapPatch`
- `MapDelete`
- `MapMethods`
- `Results`
- `EndpointRouteBuilder`
- `RazorPage`
- `PageModel`
- `OnPost`
- `OnGet`

## 2.2 RPC, realtime, function, and worker entry candidates
Search for:
- `Hub`
- `Hub<T>`
- `OnConnectedAsync`
- `SendAsync`
- `GrpcService`
- `ServiceBase`
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
- `RabbitMQ`
- `Kafka`
- `Handle`
- `Consume`
- `Import`
- `Upload`
- `Restore`
- `Replay`

## 2.3 Native and legacy formatter sink candidates
Search for:
- `BinaryFormatter`
- `BinaryFormatter.Deserialize`
- `SoapFormatter`
- `SoapFormatter.Deserialize`
- `NetDataContractSerializer`
- `ReadObject`
- `ObjectStateFormatter`
- `ObjectStateFormatter.Deserialize`
- `LosFormatter`
- `LosFormatter.Deserialize`
- `FormatterServices`
- `IFormatter`
- `SurrogateSelector`
- `ISerializationSurrogate`
- `SerializationInfo`
- `StreamingContext`
- `ISerializable`
- `IDeserializationCallback`
- `OnDeserializedAttribute`
- `OnDeserializingAttribute`
- `ViewState`
- `__VIEWSTATE`
- `EnableViewStateMac`
- `MachineKey`

## 2.4 JSON, XML, XAML, YAML, and message serializer candidates
Search for:
- `JsonConvert.DeserializeObject`
- `JsonSerializer.Deserialize`
- `JsonSerializerSettings`
- `TypeNameHandling`
- `TypeNameHandling.All`
- `TypeNameHandling.Auto`
- `TypeNameAssemblyFormatHandling`
- `SerializationBinder`
- `ISerializationBinder`
- `JavaScriptSerializer`
- `SimpleTypeResolver`
- `DataContractSerializer`
- `DataContractJsonSerializer`
- `KnownType`
- `XmlSerializer.Deserialize`
- `XmlReader`
- `XamlReader.Load`
- `XamlReader.Parse`
- `ObjectDataProvider`
- `YamlDotNet`
- `DeserializerBuilder`
- `MessagePackSerializer.Typeless`
- `TypelessContractlessStandardResolver`
- `MessagePackSerializer.Deserialize<object>`
- `protobuf-net`
- `BsonSerializer.Deserialize`
- `MongoDB.Bson.Serialization`

## 2.5 Cache, session, queue, and storage serializer candidates
Search for:
- `IDistributedCache.Get`
- `IMemoryCache.Get`
- `Session.Get`
- `TempData`
- `CookieAuthentication`
- `TicketDataFormat`
- `IDataProtector.Unprotect`
- `ProtectedData.Unprotect`
- `Convert.FromBase64String`
- `GZipStream`
- `MemoryStream`
- `Redis`
- `StackExchange.Redis`
- `EasyCaching`
- `NServiceBus`
- `MediatR`
- `MassTransit`
- `Azure.Storage.Queues`
- `ServiceBusReceivedMessage`
- `Body.ToArray`
- `message.Body`

## 2.6 Trigger and gadget behavior candidates
Search for:
- `ISerializable`
- `protected <Type>(SerializationInfo info, StreamingContext context)`
- `GetObjectData`
- `[OnDeserialized]`
- `OnDeserialization`
- `IDeserializationCallback`
- `PropertyInfo.SetValue`
- `Type.GetType`
- `Activator.CreateInstance`
- `Assembly.Load`
- `MethodInfo.Invoke`
- `Process.Start`
- `File.Delete`
- `File.WriteAllText`
- `WebClient`
- `HttpClient`
- `ObjectDataProvider`
- `ExpandedWrapper`

## 2.7 Required-control candidates
Search near sinks for:
- `TypeNameHandling.None`
- `SerializationBinder`
- `ISerializationBinder`
- `KnownTypes`
- `DataContractResolver`
- `allowed`
- `allowlist`
- `deny`
- `validate`
- `schema`
- `DataAnnotations`
- `IValidateOptions`
- `IDataProtector`
- `Protect`
- `Unprotect`
- `MachineKey.Protect`
- `EnableViewStateMac="true"`
- `ViewStateUserKey`
- `MaxDepth`
- `MaxItemsInObjectGraph`
- `XmlReaderSettings`
- `DtdProcessing.Prohibit`

## 2.8 C# / .NET graph search recipes
Useful combinations:

```text
[HttpPost] + BinaryFormatter.Deserialize
[FromBody] + JsonConvert.DeserializeObject + TypeNameHandling
MapPost + MessagePackSerializer.Typeless
HttpTrigger + JsonConvert.DeserializeObject
QueueTrigger + BinaryFormatter
ServiceBusTrigger + JsonConvert.DeserializeObject
LosFormatter + __VIEWSTATE
ObjectStateFormatter + Request
XamlReader.Load + uploaded file
JsonSerializerSettings + TypeNameHandling.All
```

---

# 3. C# / .NET Unsafe Deserialization Anti-Patterns

### A1. BinaryFormatter on request-derived data
```csharp
var formatter = new BinaryFormatter();
var obj = formatter.Deserialize(request.Body);
```

Why risky:
Untrusted input reaches legacy native object restoration.

### A2. JSON.NET TypeNameHandling with attacker-controlled JSON
```csharp
var settings = new JsonSerializerSettings { TypeNameHandling = TypeNameHandling.All };
var obj = JsonConvert.DeserializeObject<object>(json, settings);
```

Why risky:
Attacker-controlled type metadata may influence object creation unless a strict binder is used.

### A3. ViewState or LosFormatter without strong integrity
```csharp
var obj = new LosFormatter().Deserialize(viewState);
```

Why risky:
Weak ViewState integrity or leaked MachineKey can turn client state into object restoration input.

### A4. Queue message deserialized with unsafe formatter
```csharp
var obj = formatter.Deserialize(new MemoryStream(message.Body.ToArray()));
```

Why risky:
Internal queues are not automatically trusted if upstream producers can be influenced.

---

# 4. Case Templates

## Case C-DESER-1: ASP.NET endpoint to BinaryFormatter

### Vulnerable pattern
```csharp
[HttpPost]
public IActionResult Import() {
    var obj = new BinaryFormatter().Deserialize(Request.Body);
}
```

### Audit focus
Verify request reachability, formatter usage, dangerous type availability, and missing replacement with safe DTO parsing.

## Case C-DESER-2: JSON.NET polymorphic restore

### Vulnerable pattern
```csharp
JsonConvert.DeserializeObject<object>(payload, settings);
```

### Audit focus
Verify `TypeNameHandling`, binder restrictions, attacker control of `$type`, and dangerous post-restore behavior.

## Case C-DESER-3: Function or queue trigger unsafe restore

### Vulnerable pattern
```csharp
public void Run([QueueTrigger("jobs")] byte[] body) {
    formatter.Deserialize(new MemoryStream(body));
}
```

### Audit focus
Verify producer trust, message integrity, and whether queue contents can be attacker-influenced.

---

# 5. C# / .NET-Specific Audit Heuristics

## 5.1 Formatter heuristics
Treat BinaryFormatter, SoapFormatter, NetDataContractSerializer, ObjectStateFormatter, and LosFormatter as high-priority candidates when data crosses a trust boundary.

## 5.2 JSON.NET heuristics
Pay attention to `TypeNameHandling`, `$type`, binders, known-type restrictions, and wrappers hiding serializer settings.

## 5.3 ViewState heuristics
Verify ViewState MAC, ViewStateUserKey, MachineKey exposure, legacy compatibility settings, and whether user-controlled state reaches LosFormatter/ObjectStateFormatter.

## 5.4 Worker and queue heuristics
Review queue consumers, background services, function triggers, and replay tools as first-class attack surfaces, not automatically trusted internal paths.
