# C++ Path Traversal Cases

## Purpose

This file contains C++-specific path traversal, arbitrary file access, unsafe file path handling, archive extraction traversal, and local resource loading cases.

Use it when the target application includes C++ or native service code, especially in:
- HTTP, RPC, WebSocket, IPC, socket, or message-bus handlers
- custom binary protocol parsers carrying filenames or resource keys
- upload/download, import/export, backup/restore, cache, log, and file-management features
- archive extraction, plugin/resource loading, and local configuration readers

This reference is guidance, not proof. C++ path issues often combine unsafe path selection with memory-safety or privilege-boundary concerns. Always verify attacker influence, path construction, file sink behavior, and missing controls.

---

# 1. C++ Path Control Points

## 1.1 Entry and file-handling points
Look for:
- HTTP route handlers
- RPC or gRPC service methods
- socket, pipe, DBus, Binder, or custom IPC handlers
- message queue consumers
- file import/export, backup/restore, and replay paths
- local resource, plugin, config, and log loading
- archive extraction and cache cleanup

## 1.2 File operation points
Look for:
- C stdio and POSIX file APIs
- C++ iostream and filesystem APIs
- platform-specific file APIs
- Qt and Boost filesystem APIs
- archive library extraction
- dynamic library or plugin load paths

## 1.3 Path construction and validation
Look for:
- string concatenation
- filesystem path joins
- canonical/weakly canonical paths
- URL decoding
- basename/dirname handling
- symlink following
- platform separator differences

---

# 2. High-Coverage C++ Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, path construction behavior, file sink behavior, and missing containment controls.

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
- `accept`

## 2.2 Message, file, import, export, and admin entries
Search for:
- `KafkaConsumer`
- `RdKafka`
- `AMQP`
- `RabbitMQ`
- `MQTT`
- `ZeroMQ`
- `zmq_recv`
- `boost::asio`
- `import`
- `export`
- `upload`
- `download`
- `preview`
- `delete`
- `cleanup`
- `backup`
- `restore`
- `replay`
- `cache`
- `loadFromFile`
- `readFile`
- `writeFile`
- `ifstream`
- `ofstream`
- `fstream`
- `QFile`
- `QIODevice`

## 2.3 Path construction and transformation candidates
Search for:
- `std::filesystem::path`
- `std::filesystem::absolute`
- `std::filesystem::canonical`
- `std::filesystem::weakly_canonical`
- `std::filesystem::relative`
- `std::filesystem::proximate`
- `std::filesystem::path::filename`
- `std::filesystem::path::parent_path`
- `boost::filesystem::path`
- `boost::filesystem::canonical`
- `QDir`
- `QFileInfo`
- `QUrl::fromPercentEncoding`
- `QUrl::path`
- `dirname`
- `basename`
- `realpath`
- `GetFullPathName`
- `PathCchCanonicalize`
- `UrlDecode`
- `uri::decode`
- `base64_decode`
- `operator/`
- `append`
- `concat`
- `replace(\"../\"`
- `std::string`

## 2.4 File read, preview, download, resource, and plugin-load sink candidates
Search for:
- `std::ifstream`
- `std::fstream`
- `fopen`
- `open`
- `read`
- `pread`
- `mmap`
- `std::filesystem::file_size`
- `std::filesystem::exists`
- `QFile::open`
- `QFile::readAll`
- `QImage`
- `QPixmap`
- `cv::imread`
- `boost::filesystem::ifstream`
- `sendfile`
- `dlopen`
- `LoadLibrary`
- `QPluginLoader`
- `QResource`
- `readFile`
- `loadFile`

## 2.5 File write, copy, move, delete, and metadata sink candidates
Search for:
- `std::ofstream`
- `std::fstream`
- `fwrite`
- `write`
- `pwrite`
- `creat`
- `std::filesystem::copy`
- `std::filesystem::copy_file`
- `std::filesystem::rename`
- `std::filesystem::remove`
- `std::filesystem::remove_all`
- `std::filesystem::create_directory`
- `std::filesystem::create_directories`
- `std::filesystem::permissions`
- `unlink`
- `remove`
- `rename`
- `mkdir`
- `rmdir`
- `chmod`
- `chown`
- `QFile::copy`
- `QFile::rename`
- `QFile::remove`
- `QSaveFile`

## 2.6 Archive extraction candidates
Search for:
- `libzip`
- `zip_fopen`
- `zip_get_name`
- `zip_stat`
- `archive_read_next_header`
- `archive_entry_pathname`
- `archive_read_extract`
- `miniz`
- `unzOpen`
- `unzGetCurrentFileInfo`
- `QuaZip`
- `Poco::Zip`
- `boost::iostreams::gzip_decompressor`
- `tar`
- `extract`
- `unzip`
- `untar`

## 2.7 Required-control candidates
Search near sinks for:
- `std::filesystem::canonical`
- `std::filesystem::weakly_canonical`
- `std::filesystem::relative`
- `starts_with`
- `compare`
- `baseDir`
- `rootDir`
- `realpath`
- `O_NOFOLLOW`
- `openat`
- `dirfd`
- `fstatat`
- `AT_SYMLINK_NOFOLLOW`
- `is_symlink`
- `read_symlink`
- `QFileInfo::canonicalFilePath`
- `QDir::cleanPath`
- `QDir::isRelativePath`
- `allowlist`
- `validate`
- `extension`
- `mime`
- `mkstemp`
- `tmpfile`

## 2.8 C++ graph search recipes
Useful combinations:

```text
CROW_ROUTE + std::ifstream
svr.Get + fopen
grpc::Service + std::filesystem::path
recv + open
QUrl::path + QFile::open
std::filesystem::remove + request parameter
download + ifstream
upload + ofstream
archive_entry_pathname + archive_read_extract
zip_get_name + fopen/write
weakly_canonical + starts_with
```

---

# 3. C++ Path Traversal Anti-Patterns

### A1. HTTP parameter controls file read
```cpp
std::ifstream in(baseDir + "/" + req.get_param_value("file"));
```

Why risky:
User-controlled file parameters can escape the intended base directory.

### A2. Custom upload path uses original filename
```cpp
std::ofstream out(uploadDir / originalName, std::ios::binary);
```

Why risky:
Client-supplied filenames may contain traversal, absolute paths, or unsafe separators.

### A3. Unsafe delete path
```cpp
std::filesystem::remove(baseDir / name);
```

Why risky:
Weak containment allows attacker-controlled deletion outside the intended scope.

### A4. Archive entry path written directly
```cpp
auto out = dest / archive_entry_pathname(entry);
```

Why risky:
Archive entry names can escape the extraction root.

---

# 4. Case Templates

## Case CPP-PATH-1: Arbitrary read via route parameter

### Vulnerable pattern
```cpp
std::ifstream in(base / req.param("name"));
```

### Audit focus
Verify external reachability, resolved containment, symlink behavior, and disclosure impact.

## Case CPP-PATH-2: Arbitrary overwrite via upload filename

### Vulnerable pattern
```cpp
std::ofstream out(uploadDir / filename);
```

### Audit focus
Verify server-generated names, extension controls, overwrite behavior, and base containment.

## Case CPP-PATH-3: Unsafe archive extraction

### Vulnerable pattern
```cpp
std::ofstream out(dest / entryName);
```

### Audit focus
Verify every archive entry path is canonicalized under the extraction root before writing.

---

# 5. C++-Specific Audit Heuristics

## 5.1 Filesystem heuristics
Pay attention to canonicalization, weak canonicalization, symlink following, platform separators, and prefix checks.

## 5.2 POSIX and Windows API heuristics
Review `open`, `openat`, `O_NOFOLLOW`, `realpath`, `GetFullPathName`, and platform path edge cases.

## 5.3 Archive heuristics
Review archive entry pathnames, extraction destination construction, overwrite behavior, and symlink entries.

## 5.4 Plugin/resource heuristics
Treat dynamic library, plugin, resource, and config loaders as file sink candidates when paths are user-influenced.
