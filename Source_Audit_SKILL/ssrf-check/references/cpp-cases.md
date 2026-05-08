# C++ SSRF Source Cases

## Purpose

This file contains C++-specific source point patterns and candidate search terms for SSRF source discovery.

Use it when the target application includes C++ or native service code, especially in:
- HTTP, RPC, WebSocket, IPC, socket, or message-bus handlers
- libcurl, Boost.Beast/Asio, Poco, Qt Network, cpp-httplib, cpr, gRPC, raw sockets, and cloud SDK endpoint logic
- preview, import, crawler, webhook, metadata, diagnostics, proxy, browser/renderer, and native wrapper features

This reference is guidance, not proof. C++ network code often hides target construction behind wrappers. Always verify source origin, propagation, trust boundary, downstream SSRF relevance, and later target controls.

---

# 1. High-Coverage C++ SSRF Source Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; keep it only when code evidence shows that the value can influence request-target construction, URL parsing, URL recomposition, host/scheme/port/path selection, redirect behavior, DNS-sensitive target choice, proxy/client options, stored callback replay, renderer imports, cloud SDK endpoint overrides, or native outbound request wrapper behavior.

## 1.1 HTTP, RPC, WebSocket, socket, and IPC entry candidates

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
- `onOpen`
- `recv`
- `read`
- `boost::asio`
- `DBus`
- `sd_bus_message`
- `Binder`
- `onTransact`
- custom IPC handlers

## 1.2 Message, preview, import, webhook, CLI, and admin entries

Search for:
- `KafkaConsumer`
- `RdKafka`
- `AMQP`
- `RabbitMQ`
- `MQTT`
- `ZeroMQ`
- CLI argument parsing
- `argv`
- `argc`
- `getopt`
- `boost::program_options`
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
- `connectivity`
- replay handlers

## 1.3 Direct target and URL source candidates

Search for request, message, protobuf, JSON, CLI, config, or DTO fields named:
- `url`
- `uri`
- `target`
- `targetUrl`
- `requestUrl`
- `remoteUrl`
- `externalUrl`
- `callback`
- `callbackUrl`
- `webhook`
- `webhookUrl`
- `redirectUrl`
- `returnUrl`
- `previewUrl`
- `imageUrl`
- `avatarUrl`
- `fileUrl`
- `downloadUrl`
- `importUrl`
- `feedUrl`
- `sitemapUrl`
- `metadataUrl`
- `endpoint`
- `baseUrl`
- `serviceUrl`
- `providerUrl`
- `tenantUrl`

## 1.4 Partial destination, protocol, and client-option candidates

Search for:
- `host`
- `hostname`
- `domain`
- `ip`
- `address`
- `serviceName`
- `scheme`
- `protocol`
- `port`
- `path`
- `route`
- `query`
- `resource`
- `endpointOverride`
- `proxyHost`
- `proxyUrl`
- `proxy`
- `followRedirect`
- `CURLOPT_FOLLOWLOCATION`
- `CURLOPT_PROTOCOLS`
- `CURLOPT_REDIR_PROTOCOLS`
- `timeout`
- `ssl`
- `tls`
- `verifyPeer`

## 1.5 URL construction, parser, and normalization candidates

Search for source values near:
- `std::string url`
- `std::string uri`
- `std::string host`
- `std::stringstream`
- `std::ostringstream`
- `fmt::format`
- `absl::StrFormat`
- `boost::format`
- `Poco::URI`
- `QUrl`
- `QUrl::fromPercentEncoding`
- `UrlDecode`
- `base64_decode`
- `curl_url_set`
- `curl_url_get`
- `resolver.resolve`
- `getaddrinfo`
- `inet_pton`
- `boost::asio::ip::make_address`
- string concatenation around URL pieces

## 1.6 Stored, callback, and indirect fetch source candidates

Search for:
- webhook registration records
- callback target records
- integration endpoint records
- tenant endpoint settings
- provider endpoint settings
- saved crawler targets
- URL preview records
- import job URLs
- retry or replay payloads
- queue payload URLs
- remote image or avatar URLs
- OpenGraph metadata URLs
- feed or sitemap URLs
- XML/document/image renderer inputs
- CEF/WebView preview URLs
- cloud SDK endpoint overrides
- storage service endpoint settings

## 1.7 Downstream SSRF relevance mapping candidates

After finding a source candidate, trace toward:
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
- `xmlReadFile`
- `xmlReadMemory`
- `QImage`
- `cv::imread`
- AWS SDK endpoint configuration
- `Aws::Client::ClientConfiguration`
- `endpointOverride`
- shared native network wrappers

## 1.8 C++ graph search recipes

Useful combinations:

```text
CROW_ROUTE/svr.Get/ENDPOINT + url/host/endpoint + Poco::URI/QUrl/std::string
grpc::Service/ServerContext + callbackUrl/webhookUrl/previewUrl + http_client/libcurl
recv/onMessage/IPC handler + endpoint/baseUrl/providerUrl + native client wrapper
CLI argv/getopt + proxy/host/url + curl/socket/client wrapper
std::stringstream/fmt::format/QString/QUrl + request/stored host/path + outbound request
CURLOPT_FOLLOWLOCATION/CURLOPT_PROTOCOLS + request/stored URL
getaddrinfo/resolver.resolve/inet_pton + host source + connect/fetch wrapper
Aws ClientConfiguration endpointOverride + request/stored endpoint
xmlReadFile/CefBrowser/WebView + preview/import source
```

---

# 2. C++ Source Patterns

## CPP-S1. Network, RPC, IPC, or CLI source becomes request target

Example idea:
- HTTP query parameter, protobuf field, socket message, IPC payload, or CLI argument becomes a URL, host, endpoint, proxy, or raw socket target.

Audit relevance:
Native service boundaries are often custom; source classification should identify the protocol, parser, and first request-target construction point.

Follow-up:
- trace into libcurl, Boost.Beast/Asio, Poco, Qt Network, cpp-httplib, cpr, raw sockets, cloud SDKs, and wrapper services.

## CPP-S2. URL component or client option source

Example idea:
- request or stored value selects host, scheme, port, path, proxy, redirect behavior, protocol allowlist, endpoint override, or resolver target.

Audit relevance:
Partial destination control can still alter final network reachability.

Follow-up:
- verify canonical parsing, final resolved-address checks, redirect policy, protocol restrictions, and proxy constraints.

## CPP-S3. Stored callback, webhook, import, or preview source

Example idea:
- saved target, report asset, import URL, webhook callback, or queued payload later drives a native worker fetch.

Audit relevance:
Stored and background values create second-order SSRF source paths.

Follow-up:
- identify writer path and revalidation before network use.

## CPP-S4. Parser, renderer, or cloud SDK endpoint source

Example idea:
- XML/image/browser/CEF/WebView parsing or cloud SDK endpoint configuration receives weakly trusted target values.

Audit relevance:
These sources can hide outbound network behavior outside direct HTTP clients.

Follow-up:
- inspect library resource loading and endpoint override behavior.

---

# 3. False-Positive Controls

Do not mark a C++ source as high-priority if:
- the value is fixed in trusted code or selected from strict endpoint allowlists,
- the value affects only labels, logs, metrics, or non-network metadata,
- the value never reaches URL parsing, request clients, raw sockets, renderer/parser fetch behavior, endpoint builders, or network wrappers,
- wrapper internals prove the target is fixed and not attacker-influenced.

Use `Suspected source` or `Not enough evidence` if:
- protocol parsing or caller trust is unclear,
- custom network wrappers hide request-target construction,
- string construction is visible but outbound request behavior is not,
- redirects, DNS checks, proxy behavior, or stored writer paths are missing.

---

# 4. Quick C++ Source Checklist

- Do HTTP/RPC/WebSocket/IPC/socket/message/CLI inputs become URLs, hosts, endpoints, proxy settings, or request options?
- Do request or stored values influence libcurl, Boost.Asio/Beast, Poco, Qt Network, cpp-httplib, cpr, raw sockets, or cloud SDK endpoint configuration?
- Are URL strings or hosts built with string builders, formatters, `QUrl`, `Poco::URI`, or custom parsers from weakly trusted values?
- Are XML/image/browser/parser/import features fed by externally supplied targets?
- Is the source only mapped to a fixed trusted endpoint, or can it alter the final request target?
