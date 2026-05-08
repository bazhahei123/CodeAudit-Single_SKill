# C++ SSRF Cases

## Purpose

This file contains C++-specific SSRF patterns, candidate sink inventories, and audit cases.

Use it when the target application includes C++ or native service code, especially in:
- HTTP, RPC, WebSocket, IPC, socket, or message-bus handlers
- libcurl, Boost.Beast/Asio, Poco, Qt Network, cpp-httplib, gRPC, raw sockets, and cloud SDK endpoint logic
- preview, import, crawler, webhook, metadata, diagnostics, proxy, and native wrapper features

This reference is guidance, not proof. Confirm attacker influence, final request target, sink behavior, and missing destination controls.

---

# 1. C++ SSRF Control Points

## 1.1 Entry points
Look for:
- HTTP route handlers
- RPC or gRPC service methods
- socket, pipe, DBus, Binder, or custom IPC handlers
- message queue consumers
- admin, preview, import, webhook, and diagnostic flows

## 1.2 Request sinks
Look for:
- libcurl and curl wrappers
- Boost.Beast/Asio, Poco, Qt Network, cpp-httplib
- raw TCP sockets
- XML/document/image parsers loading URLs
- cloud SDK endpoint overrides

---

# 2. High-Coverage C++ SSRF Candidate Inventory

## 2.1 HTTP, RPC, WebSocket, and IPC entry candidates
Search for:
- `CROW_ROUTE`
- `crow::SimpleApp`
- `app.route_dynamic`
- `DROGON_BEGIN_NAMESPACE`
- `METHOD_LIST_BEGIN`
- `ADD_METHOD_TO`
- `HttpController`
- `HttpRequestPtr`
- `oatpp::web::server::api::Controller`
- `ENDPOINT`
- `Pistache::Rest::Routes::Post`
- `Pistache::Rest::Routes::Get`
- `httplib::Server`
- `svr.Post`
- `svr.Get`
- `grpc::Service`
- `ServerContext`
- `apache::thrift`
- `TProcessor`
- `onMessage`
- `DBus`
- `Binder`
- `onTransact`

## 2.2 Message, preview, import, webhook, and admin entries
Search for:
- `KafkaConsumer`
- `RdKafka`
- `AMQP`
- `RabbitMQ`
- `MQTT`
- `ZeroMQ`
- `boost::asio`
- `webhook`
- `callback`
- `preview`
- `fetch`
- `import`
- `download`
- `avatar`
- `image`
- `feed`
- `metadata`
- `crawler`
- `admin`
- `diagnostic`

## 2.3 C++ outbound request sink candidates
Search for:
- `curl_easy_init`
- `curl_easy_setopt`
- `CURLOPT_URL`
- `curl_easy_perform`
- `boost::beast`
- `boost::asio::ip::tcp`
- `tcp::resolver`
- `tcp::socket`
- `Poco::Net::HTTPClientSession`
- `Poco::URI`
- `QNetworkAccessManager`
- `QNetworkRequest`
- `QUrl`
- `httplib::Client`
- `cpr::Get`
- `cpr::Post`
- `web::http::client::http_client`
- `grpc::CreateChannel`
- `socket`
- `connect`

## 2.4 Parser, renderer, cloud SDK, and indirect fetch sink candidates
Search for:
- `libxml2`
- `xmlReadFile`
- `xmlReadMemory`
- `QXmlStreamReader`
- `QImage`
- `cv::imread`
- `AWS SDK`
- `Aws::Client::ClientConfiguration`
- `endpointOverride`
- `proxyHost`
- `S3Client`
- `WebView`
- `CEF`
- `CefBrowser`
- `screenshot`
- `crawler`

## 2.5 Request target construction candidates
Search for:
- `std::string url`
- `std::string uri`
- `std::string host`
- `endpoint`
- `callbackUrl`
- `webhookUrl`
- `imageUrl`
- `feedUrl`
- `metadataUrl`
- `Poco::URI`
- `QUrl`
- `UrlDecode`
- `QUrl::fromPercentEncoding`
- `base64_decode`
- `resolver.resolve`
- `getaddrinfo`
- `inet_pton`
- `proxy`

## 2.6 Required-control candidates
Search near sinks for:
- host allowlist
- scheme allowlist
- `getaddrinfo`
- `inet_pton`
- `INADDR_LOOPBACK`
- private IP check
- link-local check
- `169.254.169.254`
- `localhost`
- `127.0.0.1`
- `::1`
- `CURLOPT_FOLLOWLOCATION`
- `CURLOPT_PROTOCOLS`
- `CURLOPT_REDIR_PROTOCOLS`
- `CURLOPT_CONNECTTIMEOUT`
- `CURLOPT_TIMEOUT`
- redirect revalidation
- DNS pinning

## 2.7 C++ graph search recipes
Useful combinations:

```text
CROW_ROUTE + curl_easy_setopt CURLOPT_URL
svr.Post + httplib::Client
grpc::Service + endpoint + http_client
QUrl + QNetworkAccessManager
curl_easy_perform + user URL
CURLOPT_FOLLOWLOCATION + user URL
tcp::resolver + request host
Aws ClientConfiguration endpointOverride + user/stored value
xmlReadFile + user URL
CefBrowser + preview URL
```

---

# 3. C++ SSRF Anti-Patterns

### A1. Route parameter reaches libcurl URL
```cpp
curl_easy_setopt(curl, CURLOPT_URL, url.c_str());
curl_easy_perform(curl);
```

Why risky:
User-controlled URL may reach internal or metadata services.

### A2. Raw socket connects to user host
```cpp
resolver.resolve(host, port);
socket.connect(endpoint);
```

Why risky:
Host and port control can reach restricted networks without final IP checks.

### A3. Dynamic cloud SDK endpoint
```cpp
config.endpointOverride = endpoint;
Aws::S3::S3Client client(config);
```

Why risky:
Endpoint overrides can redirect SDK calls to attacker-selected hosts.

---

# 4. C++-Specific Audit Heuristics

Review libcurl protocol and redirect settings, resolver behavior, raw socket connection targets, Qt/Poco/Beast wrappers, XML/browser fetchers, and cloud SDK endpoint overrides.
