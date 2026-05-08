# C++ Unsafe Deserialization Source Cases

## Purpose

This file contains C++-specific source point patterns and candidate search terms for unsafe deserialization source discovery.

Use it when the target application includes C++ code, especially:
- HTTP servers and REST APIs
- CGI/FastCGI handlers
- gRPC / Thrift / custom RPC services
- WebSocket and binary protocol handlers
- native IPC services, plugins, agents, and daemons
- C++ code that receives JSON/XML/YAML/binary payloads, type registry keys, factory names, or object blobs before reconstruction

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. High-Coverage C++ Source Candidate Inventory

## 1.1 HTTP, RPC, IPC, and protocol entry candidates

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
- Netty-like channel handlers
- DBus methods
- Unix domain socket handlers
- named pipe handlers
- local TCP admin APIs
- plugin callbacks
- worker jobs
- queue consumers
- CLI/import/admin commands

## 1.2 Payload, buffer, and parser source candidates

- route captures
- query parameters
- request body
- headers
- cookies
- multipart uploads
- `std::string`
- `std::vector<uint8_t>`
- `char*`
- `unsigned char*`
- `std::istream`
- `std::stringstream`
- `std::span`
- `ByteBuffer`
- `Buffer`
- `ByteArray`
- `payload`
- `data`
- `body`
- `message`
- `blob`
- `bytes`
- `raw`
- `content`
- protobuf bytes
- IPC message fields
- environment variables in CGI

## 1.3 Serialized payload and wrapper candidates

- `serialized`
- `serializedData`
- `objectData`
- `binary`
- `base64`
- `gzip`
- `zlib`
- `zip`
- `compressed`
- `encoded`
- `encrypted`
- `signed`
- `frame`
- `packet`
- `archive`
- `metadata`
- `template`
- `config`
- `snapshot`
- `backup`
- `msgpack`
- `cereal`
- `boost`
- `protobuf`
- `flatbuffer`
- `capnp`
- `thrift`
- `yaml`
- `xml`
- `json`

## 1.4 Type, factory, and object reconstruction candidates

- `type`
- `class`
- `className`
- `objectType`
- `targetType`
- `kind`
- `discriminator`
- `tag`
- `factory`
- `factoryName`
- `registry`
- `typeRegistry`
- `create`
- `createObject`
- `fromArchive`
- `load`
- `load_construct_data`
- `serialize`
- `deserialize`
- `BOOST_CLASS_EXPORT`
- `cereal::base_class`
- `polymorphic`
- `dynamic_cast`
- `plugin`
- `handler`
- `callback`

## 1.5 Stored and second-order candidates

- cache values
- Redis values
- database blob fields
- SQLite blobs
- config files
- uploaded archives
- saved metadata
- saved templates
- saved filters
- object storage files
- IPC replay payloads
- queue messages
- dead-letter messages
- retry payloads
- plugin state files
- binary snapshots
- backup restore files
- imported project files

## 1.6 Restore and downstream mapping candidates

- `boost::archive`
- `binary_iarchive`
- `text_iarchive`
- `xml_iarchive`
- `cereal::BinaryInputArchive`
- `cereal::JSONInputArchive`
- `cereal::XMLInputArchive`
- `protobuf ParseFromString`
- `ParseFromArray`
- `ParseFromIstream`
- `MessagePack`
- `msgpack::unpack`
- `FlatBuffers`
- `capnp`
- `thrift`
- `yaml-cpp Load`
- `tinyxml`
- `pugixml`
- `nlohmann::json`
- `from_json`
- custom `deserialize`
- custom `restore`
- custom `decode`
- custom `load`

## 1.7 C++ graph search recipes

```text
HTTP/RPC/IPC handler + body/payload/buffer + base64/zlib/frame
request body + type/class/factory/registry + createObject/deserialize
queue/WebSocket/protocol frame + bytes/message + decode/restore
cache/database blob + value + boost::archive/cereal/custom deserialize
upload/import/archive/config + stream + yaml/xml/json/binary loader
protobuf/thrift/flatbuffer + bytes + ParseFromString/Deserialize
```

---

# 2. C++ Source Patterns and Blind Spots

## CPP-S1. Binary frame or buffer source

Raw frames, buffers, and stream values become source points when they reach protocol decoders, archive loaders, or object factories.

## CPP-S2. Factory/type registry source

Type names, factory keys, plugin identifiers, and registry discriminators are high-priority when they influence object construction.

## CPP-S3. Stored binary snapshot source

Project files, config backups, cache blobs, and binary snapshots can be second-order sources if an attacker can write or upload them earlier.

## CPP-S4. IPC and local admin source

Local IPC and admin tools are not automatically trusted when low-privileged users, plugins, or external agents can influence payloads.

---

# 3. False-Positive Controls

Do not mark a C++ source as high-priority if:
- the data is parsed only into strict primitives with no object/type reconstruction,
- the source is trusted-only and cannot be influenced by attacker-controlled producers,
- type registry keys are fixed server-side and not payload-controlled,
- downstream deserialization relevance is absent.

Use `Suspected source` or `Not enough evidence` if:
- parser wrappers hide object creation,
- plugin or IPC reachability is unclear,
- stored blob writer paths are missing,
- binary protocol trust assumptions are not visible.

---

# 4. Quick C++ Source Checklist

- Are HTTP, RPC, IPC, WebSocket, or binary protocol payloads decoded into objects?
- Are type, class, factory, plugin, or registry keys influenced by payloads?
- Are Boost/cereal/protobuf/thrift/msgpack/archive loaders fed by external or stored data?
- Are cache/database/project/config blobs restored later?
- Are restored fields path, URL, command, callback, handler, factory, or template values?
