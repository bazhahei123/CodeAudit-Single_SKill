# C++ Access Control Cases

## Purpose

This file contains C++-specific access control sink candidates, anti-patterns, and audit cases.

Use it when the target application includes C++ services or native components, especially:
- HTTP or REST servers implemented in C++
- gRPC services
- Qt, Boost.Beast, Crow, Drogon, Pistache, Oat++, or custom routers
- embedded admin interfaces
- local IPC, DBus, Unix sockets, named pipes, shared memory, or plugin command dispatch
- native desktop or device software exposing privileged operations

This reference is guidance, not proof. Do not report a vulnerability only because a candidate appears. Always verify the real entry point, authenticated identity, authorization check, object scoping, and privileged operation.

---

# 1. C++ Authorization Control Points

## 1.1 HTTP and REST route candidates
Look for:
- route registration tables
- `CROW_ROUTE`
- `app.route_dynamic`
- Drogon `METHOD_ADD`
- Drogon `ADD_METHOD_TO`
- Pistache `Routes::Get`, `Routes::Post`, `Routes::Put`, `Routes::Delete`
- Oat++ controllers
- Boost.Beast request handlers
- cpprestsdk `http_listener`
- Qt HTTP server routes
- custom `GET`, `POST`, `PUT`, `DELETE`, `PATCH` dispatch
- URL path parsing
- query parameter parsing

Questions:
- Which route exposes protected data or actions?
- Does the route require authentication and role/permission checks?
- Are object IDs scoped to the authenticated user or tenant?

## 1.2 Authentication and session candidates
Look for:
- session cookie parsing
- JWT validation
- OAuth token validation
- API key validation
- mTLS client certificate checks
- custom header auth
- bearer token parsing
- middleware or filter chains
- auth context stored in request objects
- global config that marks routes public

Questions:
- Can the handler run before identity is established?
- Is auth enforced consistently across equivalent routes?
- Are token claims trusted without verification or scope checks?

## 1.3 Function-level authorization candidates
Look for:
- `isAdmin`
- `hasRole`
- `hasPermission`
- `canAccess`
- `checkPermission`
- `authorize`
- `RequireRole`
- `RequirePermission`
- ACL checks
- RBAC policy lookups
- feature flag or license checks used as authorization
- admin command dispatch tables
- operation names such as `delete`, `disable`, `approve`, `export`, `reset`, `impersonate`

Questions:
- Is privileged behavior gated by a server-side authorization decision?
- Are internal/debug/admin commands reachable through external routes or IPC?
- Are equivalent handlers protected the same way?

## 1.4 Object-level and tenant candidates
Look for:
- direct lookup by `id`
- repository methods such as `getById`, `findById`, `load`, `fetch`
- SQL queries missing user or tenant predicates
- object maps keyed by externally supplied IDs
- file/download IDs
- tenant, org, account, project, or workspace IDs from request input
- cached objects returned before authorization
- update/delete operations after unscoped lookup

Questions:
- Is the object bound to the authenticated principal or trusted tenant context?
- Is tenant scope derived from server-side context rather than user input?
- Are bulk/export paths scoped like single-object paths?

## 1.5 RPC, IPC, and native boundary candidates
Look for:
- gRPC service implementations
- protobuf service methods
- Thrift handlers
- Cap'n Proto services
- DBus interfaces
- Unix domain sockets
- named pipes
- local TCP admin ports
- shared-memory command queues
- plugin dispatch tables
- JNI native methods called from Android/Java
- command handlers registered by string name

Questions:
- Who can connect or call this interface?
- Is caller identity checked per method?
- Are privileged commands restricted by role, owner, tenant, or device state?

## 1.6 High-coverage C++ sink candidate inventory

Search candidates:
- `CROW_ROUTE`
- `METHOD_ADD`
- `ADD_METHOD_TO`
- `Routes::Get`
- `Routes::Post`
- `Routes::Put`
- `Routes::Delete`
- `http_listener`
- `handle_get`
- `handle_post`
- `onRequest`
- `doGet`
- `doPost`
- `grpc::Server`
- `Service::AsyncService`
- `RequestHandler`
- `authorize`
- `checkPermission`
- `hasPermission`
- `hasRole`
- `isAdmin`
- `getCurrentUser`
- `session`
- `jwt`
- `apiKey`
- `tenantId`
- `ownerId`
- `findById`
- `getById`
- `deleteById`
- `export`
- `download`
- `admin`
- `debug`
- `impersonate`

---

# 2. C++ Access Control Anti-Patterns

## A1. Handler performs privileged action without authz check
Why risky:
Native services often use custom routing, so missing checks are easy to miss.

Verify:
- request identity establishment,
- role or permission check,
- object owner or tenant binding.

## A2. Local IPC assumes local means trusted
Why risky:
Local users, sandboxed apps, or compromised lower-privilege processes may access privileged IPC.

Verify:
- socket permissions,
- peer credentials,
- UID/GID checks,
- per-method authorization.

## A3. Object lookup by raw external ID
Why risky:
Direct lookup by ID without owner or tenant scope may create object-level authorization bypass.

Verify:
- trusted principal context,
- scoped query,
- check before return/update/delete.

## A4. Debug or admin endpoint enabled in production
Why risky:
Admin handlers may expose export, config, user-management, or impersonation behavior.

Verify:
- route exposure,
- environment gating,
- authentication and authorization.

---

# 3. Quick C++ Audit Checklist

- Are HTTP/RPC/IPC handlers authenticated before sensitive logic?
- Are admin and debug commands authorization-checked?
- Are object lookups scoped by trusted identity or tenant?
- Are local IPC interfaces protected by peer credential checks?
- Are gRPC/Thrift service methods checked individually?
- Are bulk/export/download paths scoped like normal reads?
- Are route dispatch tables hiding alternate unprotected handlers?
