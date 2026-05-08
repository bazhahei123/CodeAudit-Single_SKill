# C++ Access Control Source Cases

## Purpose

This file contains C++-specific source point patterns and audit cases for access-control source discovery.

Use it when the target application includes C++ services or native components, especially:
- HTTP servers and REST APIs
- CGI/FastCGI modules
- RPC/gRPC/Thrift services
- WebSocket handlers
- desktop/native applications with IPC or embedded web views
- native plugins, daemons, agents, and service boundaries
- C++ code that forwards identity, tenant, object, role, or workflow values into protected operations

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real source origin and downstream use in the target code.

---

# 1. High-Coverage C++ Source Candidate Inventory

## 1.1 HTTP, REST, and web entry candidates

Look for:
- cpp-httplib route handlers: `Get`, `Post`, `Put`, `Patch`, `Delete`
- Crow routes: `CROW_ROUTE`, `.methods(...)`
- Drogon controllers: `HttpController`, `METHOD_LIST_BEGIN`, `ADD_METHOD_TO`, `PATH_ADD`, `METHOD_ADD`
- Pistache routes and handlers
- oatpp controllers: `ENDPOINT`, `ENDPOINT_ASYNC`
- Restbed resources and handlers
- Wt resources, request handlers, and server entry points
- CGI/FastCGI entry points: `FCGX_Accept`, environment variables, query strings, POST bodies
- custom routers, switch/case dispatchers, admin endpoints, webhook handlers

Source questions:
- Which handlers receive external identifiers such as `id`, `user_id`, `tenant_id`, `org_id`, `file_id`?
- Which handlers expose sensitive actions such as export, delete, approve, publish, reset, refund, or disable?
- Are the same service methods reachable through HTTP, RPC, IPC, CLI, or background jobs?

## 1.2 Request parsing and client-controlled source candidates

Look for:
- URL path captures and route parameters
- query string parsing
- JSON body parsing with nlohmann/json, RapidJSON, JsonCpp, Boost.JSON
- form/body parsers
- HTTP headers and cookies
- multipart file metadata
- protobuf/thrift request fields
- WebSocket JSON messages
- command-line arguments for user-triggered tools
- environment variables in CGI/FastCGI
- custom `Request`, `Context`, `Session`, or `Params` wrappers

High-priority fields:
- `id`, `ids`, `user_id`, `userId`, `uid`, `account_id`, `owner_id`
- `tenant_id`, `tenantId`, `org_id`, `workspace_id`, `company_id`
- `role`, `roles`, `permission`, `scope`, `is_admin`, `access_level`
- `order_id`, `invoice_id`, `file_id`, `project_id`, `document_id`, `payment_id`
- `action`, `operation`, `status`, `state`, `target_state`

## 1.3 Authentication identity source candidates

Look for:
- request/session context objects carrying current user
- JWT parsing and claim extraction: `sub`, `user_id`, `uid`, `tenant_id`, `scope`, `roles`
- OAuth/OIDC middleware context
- mTLS peer certificate subject or SAN values
- API key lookup results
- cookie/session stores
- custom helpers such as `currentUser`, `getCurrentUser`, `UserContext`, `AuthContext`, `Principal`, `TenantContext`
- thread-local or request-local context values

Source questions:
- Is identity derived from verified middleware, or only decoded from a token/header?
- Can request fields override session or principal values?
- Is tenant or organization membership loaded server-side?

## 1.4 Role, permission, policy, and authority candidates

Look for:
- `authorize`
- `authenticate`
- `requireAuth`
- `requireRole`
- `requirePermission`
- `checkPermission`
- `canAccess`
- `canRead`
- `canWrite`
- `canDelete`
- `isAdmin`
- `isOwner`
- `isTenantMember`
- `acl`
- `rbac`
- `policy`
- `AccessControl`
- Casbin, OPA, custom policy engines
- route metadata or middleware that binds roles, scopes, or permissions

## 1.5 Object, tenant, database, and repository source candidates

Look for:
- object IDs: `id`, `userId`, `ownerId`, `accountId`, `orderId`, `invoiceId`, `fileId`, `projectId`, `documentId`, `resourceId`, `paymentId`
- tenant scopes: `tenantId`, `orgId`, `organizationId`, `companyId`, `workspaceId`, `teamId`, `departmentId`, `accountId`
- repository methods: `getById`, `findById`, `loadById`, `deleteById`, `getForUser`, `getForTenant`, `exportForOrg`
- ORM/query APIs: SOCI, sqlpp11, SQLite wrappers, ODB, Qt SQL, `exec`, `prepare`, `bind`, `where`
- database filters using owner, tenant, account, workspace, role, or status values
- batch collections such as `std::vector<int> ids`, `std::vector<std::string> ids`, repeated protobuf fields

## 1.6 RPC, IPC, WebSocket, plugin, and async entry candidates

Look for:
- gRPC service methods and protobuf request fields
- Thrift service handlers
- Cap'n Proto, ZeroMQ, custom binary RPC handlers
- DBus methods, Qt signals/slots exposed across process boundaries
- named pipes, Unix domain sockets, local TCP admin APIs
- WebSocket message handlers
- plugin callbacks and native extension entry points
- queue consumers and worker jobs when payloads originate from user-triggered actions
- desktop embedded web view bridge handlers such as Qt WebChannel, CEF message routers, WebView2 host objects

## 1.7 Business action and workflow candidates

Look for function, route, message, or enum names containing:
- `approve`
- `reject`
- `publish`
- `archive`
- `delete`
- `disable`
- `enable`
- `lock`
- `unlock`
- `reset`
- `refund`
- `void`
- `cancel`
- `transfer`
- `assign`
- `share`
- `export`
- `download`
- `invite`
- `promote`
- `demote`
- `grant`
- `revoke`

Also inspect fields:
- `action`
- `operation`
- `status`
- `state`
- `stage`
- `transition`
- `targetState`
- `targetStatus`

---

# 2. C++ Source Patterns and Blind Spots

## 2.1 HTTP object source

```cpp
server.Get(R"(/orders/(\d+))", [](const Request& req, Response& res) {
    auto orderId = req.matches[1];
    res.set_content(orderService.get(orderId), "application/json");
});
```

Why relevant:
`orderId` is client-controlled and selects a protected object.

What to verify next:
- whether current user or tenant context is used downstream
- whether the route is public or authenticated
- whether equivalent RPC/IPC paths reach the same service

## 2.2 Request tenant source

```cpp
auto tenantId = request.query("tenant_id");
auto report = reportService.exportReport(tenantId, reportId);
```

Why relevant:
`tenant_id` is client-controlled and may influence cross-tenant export behavior.

What to verify next:
- whether tenant membership is loaded from trusted context
- whether request tenant values are treated only as selectors
- whether policy or repository code rechecks tenant scope

## 2.3 IPC action source

```cpp
void AdminService::RunAction(const Message& msg) {
    auto projectId = msg.get("project_id");
    auto action = msg.get("action");
    projectService.run(projectId, action);
}
```

Why relevant:
IPC message fields can be client-controlled source points for protected local or backend operations.

What to verify next:
- whether caller identity is authenticated
- whether local process/user permissions are checked
- whether the service validates role, object, tenant, and workflow state

---

# 3. C++ Graph Search Recipes

```text
HTTP route handler + query/path/body parser + id/tenant_id/action/status + service/repository call
JWT/session/API key context + object id/tenant id + policy/database call
gRPC/Thrift/IPC method + request field user_id/tenant_id/object_id + protected operation
WebSocket/plugin/native bridge + message field action/status + service method
batch vector/repeated field ids + delete/export/share/approve action
```

---

# 4. False-Positive Controls

Do not mark a C++ source as high-priority if:
- the value does not reach protected object access, tenant scope, policy logic, or privileged workflow,
- the request value is overwritten by trusted identity, membership, or tenant context before sensitive use,
- the endpoint or IPC method is not externally reachable and no indirect trigger is visible,
- the value only affects pagination, sorting, display, or non-sensitive filtering.

Use `Suspected source` or `Not enough evidence` if:
- authentication middleware or request context population is hidden,
- caller identity for IPC/RPC is unclear,
- service/repository code is missing,
- route registration or plugin reachability is unclear.

---

# 5. Quick C++ Source Checklist

- Are object IDs read from HTTP path/query/body, RPC fields, IPC messages, WebSocket messages, or plugin callbacks?
- Are `user_id`, `tenant_id`, `org_id`, `role`, `permission`, or `scope` accepted from external input?
- Is current identity read from verified session/JWT/mTLS/API key context rather than raw headers?
- Are policy helpers passed both trusted and client-controlled values?
- Are repository/database calls scoped by trusted user or tenant values?
- Are native IPC, local admin APIs, CLI tools, or worker paths exposing equivalent protected actions?
