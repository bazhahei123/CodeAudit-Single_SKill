# C# and .NET SQL Injection Source Cases

## Purpose

This file contains C#/.NET-specific source point patterns and candidate search terms for SQL injection source discovery.

Use it when the target application is implemented in C# or .NET, especially in:
- ASP.NET MVC, Web API, Razor Pages, minimal APIs, and ASP.NET Core
- SignalR hubs, gRPC services, WCF services, Azure Functions, queue consumers, and hosted workers
- ADO.NET, Dapper, EF Core, LINQ-to-SQL, NHibernate, and database client wrappers
- admin panels, reporting systems, exports, dashboards, search, filter, sort, saved query, and analytics features

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify real source origin, propagation, trust boundary, downstream SQL relevance, and later binding or structural controls.

---

# 1. High-Coverage C#/.NET SQL Source Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; keep it only when code evidence shows that the value can influence SQL values, SQL structure, raw fragments, ORM/query-builder input, stored query metadata, or database wrapper behavior.

## 1.1 ASP.NET, Web API, Razor Pages, and minimal API entry candidates

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
- `Request.Query`
- `Request.Form`
- `Request.Cookies`
- `Request.Headers`
- `MapGet`
- `MapPost`
- `MapPut`
- `MapPatch`
- `MapDelete`
- `MapGroup`
- `PageModel`
- `OnGet`
- `OnPost`
- `BindProperty`

## 1.2 RPC, function, worker, report, and admin entries

Search for:
- `Hub`
- `Hub<T>`
- `GrpcService`
- `ServerCallContext`
- `[OperationContract]`
- `[ServiceContract]`
- `[WebMethod]`
- `HttpTrigger`
- `QueueTrigger`
- `ServiceBusTrigger`
- `TimerTrigger`
- `IHostedService`
- `BackgroundService`
- `ExecuteAsync`
- `Hangfire`
- `IConsumer`
- `Handle`
- `Consume`
- `MediatR`
- `IRequestHandler`
- `Admin`
- `Dashboard`
- `Report`
- `Export`
- `Search`
- `Filter`
- `Sort`
- `SavedQuery`
- import handlers
- replay handlers

## 1.3 SQL value and criteria source candidates

Search for route, query, body, DTO, command, message, or model fields named:
- `q`
- `query`
- `keyword`
- `search`
- `term`
- `username`
- `email`
- `status`
- `state`
- `type`
- `category`
- `tenantId`
- `accountId`
- `userId`
- `orgId`
- `ids`
- `from`
- `to`
- `startDate`
- `endDate`
- `filters`
- `criteria`
- `conditions`
- `rules`
- `predicate`
- `where`
- `specification`

## 1.4 Structural SQL selector source candidates

Search for:
- `sort`
- `sortBy`
- `orderBy`
- `direction`
- `field`
- `fields`
- `column`
- `columns`
- `select`
- `projection`
- `table`
- `tableName`
- `schema`
- `database`
- `partition`
- `operator`
- `op`
- `comparator`
- `join`
- `groupBy`
- `having`
- `limit`
- `offset`
- `page`
- `pageSize`
- `cursor`
- `top`
- `skip`
- `take`
- `procedure`
- `procedureName`

## 1.5 Raw fragment and query template source candidates

Search for:
- `sql`
- `rawSql`
- `querySql`
- `nativeQuery`
- `customQuery`
- `reportSql`
- `whereClause`
- `orderClause`
- `havingClause`
- `joinClause`
- `condition`
- `expression`
- `filterExpression`
- `queryTemplate`
- `reportTemplate`
- `savedQuery`
- `dashboardQuery`
- `StringBuilder`
- `string.Format`
- `$"{`
- `FormattableString`
- `RawSqlString`
- concatenation around `WHERE`, `ORDER BY`, `LIMIT`, `OFFSET`, `SELECT`, `FROM`, or `JOIN`

## 1.6 ORM, query builder, and wrapper input candidates

Search for source values passed into:
- ADO.NET command wrapper methods
- `SqlCommand.CommandText`
- `DbCommand.CommandText`
- EF Core `FromSqlRaw`
- EF Core `ExecuteSqlRaw`
- EF Core `SqlQueryRaw`
- EF Core `FromSqlInterpolated`
- Dapper `.Query`
- Dapper `.QueryAsync`
- Dapper `.Execute`
- Dapper `.ExecuteAsync`
- `DynamicParameters`
- NHibernate `CreateSQLQuery`
- `System.Linq.Dynamic`
- `OrderBy(string`
- `Where(string`
- `DynamicExpressionParser`
- stored procedure name variables
- report/query/export services
- saved filter readers and query metadata loaders

## 1.7 Downstream SQL relevance mapping candidates

After finding a source candidate, trace toward:
- `SqlCommand`
- `DbCommand`
- `IDbCommand`
- `CommandText`
- `ExecuteReader`
- `ExecuteScalar`
- `ExecuteNonQuery`
- `SqlDataAdapter`
- `OleDbCommand`
- `OdbcCommand`
- `NpgsqlCommand`
- `MySqlCommand`
- `SqliteCommand`
- `OracleCommand`
- `FromSqlRaw`
- `ExecuteSqlRaw`
- `Database.SqlQueryRaw`
- Dapper query/execute calls
- stored procedure callers
- repository, report, export, analytics, or dashboard query helpers

## 1.8 C#/.NET graph search recipes

Useful combinations:

```text
[HttpGet]/[HttpPost] + [FromQuery]/[FromBody] filter/sort + repository/query helper
MapGet/MapPost + orderBy/field + CommandText/FromSqlRaw/Dapper
HttpTrigger/QueueTrigger + savedFilter/reportTemplate + query builder/helper
SignalR/gRPC/WCF method + criteria/whereClause + database wrapper
StringBuilder/string.Format/$ interpolation + request/stored field + WHERE/ORDER BY/LIMIT
System.Linq.Dynamic OrderBy/Where + request DTO field
```

---

# 2. C#/.NET Source Patterns

## C-S1. Request or DTO value becomes query criteria

Example idea:
- query string, body DTO, route value, SignalR argument, gRPC message, or function payload becomes a repository, Dapper, EF, or ADO.NET query argument.

Audit relevance:
The source may be safe only if it is bound as a value or passed through a safe typed LINQ expression with no raw structural influence.

Follow-up:
- trace into EF Core, Dapper, ADO.NET, NHibernate, repository wrappers, and report generators.

## C-S2. Dynamic structural selector source

Example idea:
- request or stored value selects sort field, direction, column, table, procedure, grouping, or dynamic LINQ expression.

Audit relevance:
Parameters do not protect most structural SQL choices. Dynamic LINQ and string-based ordering also need mapping review.

Follow-up:
- verify fixed mappings, enums, expression tree construction, and structural allowlists.

## C-S3. Raw SQL text or template source

Example idea:
- `whereClause`, `reportSql`, `customQuery`, or interpolated string content reaches `CommandText`, EF raw SQL, Dapper, or a wrapper.

Audit relevance:
Raw fragments are high-priority when they originate from requests, queues, saved filters, report templates, or admin input.

Follow-up:
- inspect binding, allowlists, `FormattableString` semantics, and wrapper internals.

## C-S4. Stored, worker, or report source

Example idea:
- saved query metadata, report templates, queue messages, timer jobs, or background workers assemble query options later.

Audit relevance:
Stored values require writer-path review and revalidation before query use.

Follow-up:
- identify who can write the stored value and whether it is structured or raw SQL.

---

# 3. False-Positive Controls

Do not mark a C#/.NET source as high-priority if:
- the value is fixed server-side or selected from a strict enum/allowlist,
- the value is used only as a real `DbParameter`, Dapper anonymous parameter, or typed LINQ value with no structural influence,
- dynamic strings are not used in SQL, dynamic LINQ, raw helpers, or database wrappers,
- stored query metadata is trusted-only and not attacker-influenced.

Use `Suspected source` or `Not enough evidence` if:
- repository/helper behavior is hidden,
- EF Core or Dapper API semantics are unclear from the call site,
- `FormattableString` vs raw string behavior is ambiguous,
- stored filter writer paths are missing.

---

# 4. Quick C#/.NET Source Checklist

- Are route, body, query, SignalR, gRPC, Azure Function, queue, or worker values used as query criteria?
- Do request or stored values influence `CommandText`, EF raw SQL, Dapper SQL strings, dynamic LINQ, stored procedure names, or report query builders?
- Are sort fields, table names, columns, operators, directions, limit/offset/top values, and raw conditions mapped to fixed choices?
- Are saved filters, report templates, dashboard definitions, or queued query options revalidated before query construction?
- Is the source only a bound value, or can it change SQL structure?
