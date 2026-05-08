# C++ Unsafe Deserialization Cases

## Purpose

This file contains C++-specific unsafe deserialization patterns, candidate sink inventories, and audit cases.

Use it when the target application includes C++ or native service code, especially in:
- HTTP, RPC, WebSocket, IPC, socket, or message-bus handlers
- custom binary protocol parsers
- Boost.Serialization, cereal, Qt, MFC, protobuf-like, MessagePack, CBOR, BSON, YAML, XML, and JSON restoration paths
- native import, replay, backup, restore, or cache-loading utilities
- object factories, plugin registries, class-name-based construction, or polymorphic archive loaders

This reference is guidance, not proof. C++ deserialization issues often combine object integrity failure with memory-safety, parser, allocation, or type-confusion risk. Always verify attacker influence, parser behavior, object construction, post-restore effects, and missing controls.

---

# 1. C++ Deserialization Control Points

## 1.1 Entry and restore points
Look for:
- HTTP route handlers
- RPC or gRPC service methods
- socket, pipe, DBus, Binder, or custom IPC handlers
- message queue consumers
- file import, restore, replay, and cache-loading paths
- binary archives and stream extraction operators
- object factories driven by type names or IDs

## 1.2 Object behavior controls
Look for:
- `serialize`, `load`, `save`, `load_construct_data`
- stream extraction operators
- factory registry creation
- virtual dispatch after hydration
- callbacks, destructors, and RAII side effects
- dynamic casts and type confusion after parse

## 1.3 Framework and library controls
Look for:
- Boost.Serialization
- cereal archives
- Qt `QDataStream`
- MessagePack, CBOR, BSON, YAML, XML, and JSON parsers
- protobuf/thrift/flatbuffers/capnproto schema handling
- custom binary decoders
- size, depth, and version controls

## 1.4 Trust-boundary controls
Look for:
- signed serialized blobs
- trusted producer guarantees
- schema validation
- type ID allowlists
- max length/depth checks
- integer overflow and allocation checks
- internal-only protocol assumptions

---

# 2. High-Coverage C++ Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, sink behavior, trigger behavior, and missing controls.

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
- `web::http::experimental::listener::http_listener`
- `grpc::Service`
- `ServerContext`
- `Request`
- `Response`
- `apache::thrift`
- `TProcessor`
- `onMessage`
- `websocket`
- `DBus`
- `sd_bus_message`
- `Binder`
- `onTransact`
- `read`
- `recv`
- `recvfrom`
- `accept`
- `socket`
- `pipe`

## 2.2 Message, file, import, and replay entry candidates
Search for:
- `KafkaConsumer`
- `RdKafka`
- `AMQP`
- `RabbitMQ`
- `MQTT`
- `ZeroMQ`
- `zmq_recv`
- `libuv`
- `boost::asio`
- `import`
- `upload`
- `restore`
- `replay`
- `sync`
- `backup`
- `cache`
- `loadFromFile`
- `readFile`
- `ifstream`
- `istream`
- `mmap`
- `fread`
- `readAll`
- `QFile`
- `QIODevice`

## 2.3 Archive and native object-restoration sink candidates
Search for:
- `boost::archive::binary_iarchive`
- `boost::archive::text_iarchive`
- `boost::archive::xml_iarchive`
- `operator>>`
- `BOOST_SERIALIZATION`
- `BOOST_CLASS_EXPORT`
- `BOOST_CLASS_EXPORT_KEY`
- `BOOST_SERIALIZATION_ASSUME_ABSTRACT`
- `serialize(Archive`
- `template<class Archive>`
- `load(Archive`
- `save(Archive`
- `split_member`
- `load_construct_data`
- `cereal::BinaryInputArchive`
- `cereal::JSONInputArchive`
- `cereal::XMLInputArchive`
- `CEREAL_REGISTER_TYPE`
- `CEREAL_REGISTER_POLYMORPHIC_RELATION`
- `deserialize`
- `unserialize`
- `restore`
- `hydrate`
- `fromBinary`
- `readObject`

## 2.4 Structured parser and binary decoder candidates
Search for:
- `google::protobuf::Message`
- `ParseFromString`
- `ParseFromArray`
- `ParseFromIstream`
- `MergeFromString`
- `SerializeAsString`
- `flatbuffers::Verifier`
- `GetRoot`
- `capnp::FlatArrayMessageReader`
- `apache::thrift::TDeserializer`
- `msgpack::unpack`
- `msgpack::object`
- `nlohmann::json::parse`
- `from_json`
- `rapidjson::Document::Parse`
- `YAML::Load`
- `YAML::LoadFile`
- `tinyxml2::XMLDocument::Parse`
- `pugi::xml_document::load`
- `QDataStream`
- `QVariant`
- `QJsonDocument::fromJson`
- `QCborValue::fromCbor`
- `bsoncxx::from_json`
- `CBOR`
- `BSON`

## 2.5 Dynamic type, factory, and plugin construction candidates
Search for:
- `typeid`
- `typeName`
- `className`
- `class_id`
- `factory`
- `registry`
- `create`
- `createInstance`
- `newInstance`
- `dynamic_cast`
- `static_cast`
- `reinterpret_cast`
- `std::function`
- `callback`
- `virtual`
- `dlopen`
- `dlsym`
- `LoadLibrary`
- `GetProcAddress`
- `QMetaObject`
- `QMetaType`
- `QMetaObject::newInstance`
- `QObject`

## 2.6 Memory-safety and resource trigger candidates
Search near parsers for:
- `length`
- `size`
- `count`
- `capacity`
- `reserve`
- `resize`
- `new`
- `malloc`
- `calloc`
- `realloc`
- `memcpy`
- `memmove`
- `strcpy`
- `strncpy`
- `read`
- `seekg`
- `tellg`
- `uint32_t`
- `int`
- `overflow`
- `recursion`
- `depth`
- `while`
- `for`

## 2.7 Required-control candidates
Search near sinks for:
- `schema`
- `version`
- `magic`
- `allowlist`
- `whitelist`
- `typeMap`
- `factoryMap`
- `maxSize`
- `maxLength`
- `maxDepth`
- `limit`
- `bounds`
- `checked`
- `Verifier`
- `Verify`
- `IsInitialized`
- `has_`
- `validate`
- `signature`
- `HMAC`
- `MAC`
- `hash`
- `constant time`
- `trusted`
- `capability`
- `permission`
- `sandbox`
- `fuzz`

## 2.8 C++ graph search recipes
Useful combinations:

```text
CROW_ROUTE + boost::archive::binary_iarchive
svr.Post + deserialize
grpc::Service + ParseFromString + factory
recv + cereal::BinaryInputArchive
ifstream + boost::archive::text_iarchive
QDataStream + operator>>
msgpack::unpack + typeName
YAML::Load + createInstance
nlohmann::json::parse + from_json + factory
length + resize + memcpy
BOOST_CLASS_EXPORT + untrusted archive
```

---

# 3. C++ Unsafe Deserialization Anti-Patterns

### A1. Binary archive restored from request or socket data
```cpp
boost::archive::binary_iarchive archive(stream);
archive >> obj;
```

Why risky:
Untrusted data can influence object graph restoration, allocation, or polymorphic type reconstruction.

### A2. Type name from input drives object factory
```cpp
auto obj = registry.create(parsed["type"].get<std::string>());
obj->load(parsed);
```

Why risky:
Attacker-controlled type selection may instantiate unexpected classes or trigger unsafe post-restore behavior.

### A3. Custom binary parser trusts length fields
```cpp
uint32_t len = read_u32(buf);
std::vector<char> out(len);
memcpy(out.data(), ptr, len);
```

Why risky:
C++ deserialization commonly becomes exploitable through missing size, bounds, depth, or allocation controls.

### A4. Internal queue or file import assumes trusted producer
```cpp
auto msg = consumer.receive();
deserialize(msg.payload(), state);
```

Why risky:
Internal message transport does not prove trusted origin when upstream input can be attacker-influenced.

---

# 4. Case Templates

## Case CPP-DESER-1: HTTP endpoint to archive restore

### Vulnerable pattern
```cpp
void importHandler(const Request& req) {
    std::stringstream ss(req.body);
    boost::archive::binary_iarchive ar(ss);
    ar >> obj;
}
```

### Audit focus
Verify public reachability, input control, archive type restrictions, object behavior, and size/depth controls.

## Case CPP-DESER-2: Polymorphic factory from parsed type

### Vulnerable pattern
```cpp
auto type = doc["type"].GetString();
auto obj = factory.create(type);
obj->hydrate(doc);
```

### Audit focus
Verify type allowlists, allowed classes, post-restore side effects, and whether the type field is attacker-controlled.

## Case CPP-DESER-3: Message consumer custom binary decoder

### Vulnerable pattern
```cpp
consumer.onMessage([](auto body) {
    State s;
    deserialize(body.data(), body.size(), s);
});
```

### Audit focus
Verify producer trust, parser bounds checks, memory-safety controls, and consistency with public import paths.

---

# 5. C++-Specific Audit Heuristics

## 5.1 Archive heuristics
Review Boost.Serialization and cereal uses for polymorphic type handling, exported classes, object graph reconstruction, and untrusted archives.

## 5.2 Custom parser heuristics
Treat length fields, recursion, allocation, pointer arithmetic, and stream extraction as core deserialization review points.

## 5.3 Factory heuristics
Look for class names, type IDs, plugin registries, and dynamic construction based on parsed data.

## 5.4 Protocol consistency heuristics
Compare HTTP, RPC, IPC, queue, file import, and replay paths for inconsistent validation before object restoration.
