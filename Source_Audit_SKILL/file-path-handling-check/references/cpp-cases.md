# C++ File Path Handling Source Cases

## Purpose

This file contains C++-specific source point patterns and candidate search terms for path traversal and unsafe file path handling source discovery.

Use it when the target application includes C++ code, especially:
- HTTP servers and REST APIs
- CGI/FastCGI handlers
- gRPC / Thrift / custom RPC services
- WebSocket and binary protocol handlers
- native IPC services, plugins, agents, and daemons
- file import/export, archive extraction, template/resource loading, cleanup, and native storage workflows

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. High-Coverage C++ Source Candidate Inventory

## 1.1 HTTP, RPC, IPC, and native entry candidates

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
- DBus methods
- Unix domain socket handlers
- named pipe handlers
- local TCP admin APIs
- plugin callbacks
- queue consumers
- worker jobs
- CLI/import/admin commands

## 1.2 Payload and path-like source candidates

- route captures
- query parameters
- request body
- headers
- cookies
- multipart upload names
- protobuf string fields
- IPC message fields
- CLI args
- environment variables in CGI
- `file`
- `filename`
- `fileName`
- `path`
- `filePath`
- `dir`
- `directory`
- `folder`
- `resource`
- `template`
- `config`
- `log`
- `storageKey`
- `objectKey`
- `uri`
- `url`
- `export`
- `destination`
- `target`
- `cleanupTarget`

## 1.3 Path construction and transform candidates

- `std::filesystem::path`
- `std::filesystem::absolute`
- `std::filesystem::canonical`
- `std::filesystem::weakly_canonical`
- `std::filesystem::relative`
- `operator/`
- `append`
- `lexically_normal`
- `boost::filesystem::path`
- `realpath`
- `basename`
- `dirname`
- `PathCombine`
- `GetFullPathName`
- `std::ifstream`
- `std::ofstream`
- `fopen`
- `open`
- `stat`
- `unlink`
- `rename`
- `remove`

## 1.4 Upload, archive, and import source candidates

- multipart filename
- content-disposition filename
- upload metadata
- `ZipArchive`
- `libzip`
- `minizip`
- `archive_entry_pathname`
- `libarchive`
- tar entry names
- package manifest paths
- backup file paths
- restore manifests
- imported project files
- plugin package entries
- extraction destination

## 1.5 Stored, queue, and second-order candidates

- database path fields
- SQLite path fields
- config file paths
- project file paths
- cache values
- queue message payloads
- retry payloads
- job args
- plugin state files
- export paths
- cleanup targets
- object storage keys
- saved reports
- saved templates
- downloaded metadata

## 1.6 Downstream mapping candidates

- `std::ifstream`
- `std::ofstream`
- `std::fstream`
- `fopen`
- `open`
- `read`
- `write`
- `remove`
- `unlink`
- `rename`
- `std::filesystem::remove`
- `std::filesystem::copy`
- `std::filesystem::rename`
- `std::filesystem::create_directories`
- `QFile`
- `QDir`
- `QResource`
- `QFileInfo`
- archive extraction writes
- template/resource loaders

## 1.7 C++ graph search recipes

```text
HTTP/RPC/IPC handler + query/body file/path/name + filesystem::path/absolute/canonical
multipart/upload metadata + filename + ofstream/fopen/copy
archive_entry_pathname/ZipArchive + entry name + extraction write
queue/job/plugin payload + path/file/exportPath + remove/rename/copy
template/resource/config selector + QResource/QFile/ifstream
database/project metadata + stored path + read/delete/cleanup
```

---

# 2. C++ Source Patterns and Blind Spots

## CPP-S1. Request and protocol path values

HTTP, RPC, IPC, WebSocket, and CLI inputs become source points when they are converted into filesystem paths or resource selectors.

## CPP-S2. Multipart and archive metadata

Upload filenames, package manifests, tar paths, and zip entry names are path sources for write and extraction logic.

## CPP-S3. Native plugin and local admin sources

Plugin callbacks, local admin commands, and IPC payloads are not automatically trusted when lower-privileged users or external components can influence them.

## CPP-S4. Stored project/config path sources

Project files, config records, database fields, and cached metadata can become second-order path sources.

---

# 3. False-Positive Controls

Do not mark a C++ source as high-priority if:
- the value is selected from a strict allowlist of safe resource keys,
- uploaded filenames are replaced by generated safe names,
- path-like values are display-only and never reach file/resource operations,
- the source is trusted-only and cannot be influenced by external or lower-privileged producers.

Use `Suspected source` or `Not enough evidence` if:
- native wrapper helpers hide canonical containment checks,
- plugin/IPC reachability is unclear,
- archive extraction behavior is abstracted,
- stored path writer paths are missing.

---

# 4. Quick C++ Source Checklist

- Are HTTP, RPC, IPC, CLI, or plugin values used as filenames, paths, templates, or storage keys?
- Are upload filenames or archive entry names used for write/extraction destinations?
- Are database/project/config/cache path fields later used for reads or cleanup?
- Are symlinks, canonical paths, and alternate separators considered in follow-up checks?
- Are delete, move, copy, export, cleanup, and read operations sourced differently?
