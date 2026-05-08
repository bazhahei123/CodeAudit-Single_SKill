# C# and .NET SQL Injection Cases

## Purpose

This file contains C#/.NET-specific SQL injection patterns, candidate sink inventories, and audit cases.

Use it when the target application is primarily implemented in C# or .NET, especially in:
- ASP.NET MVC, Web API, Razor Pages, minimal APIs, and ASP.NET Core
- SignalR hubs, gRPC services, WCF services, Azure Functions, and background workers
- ADO.NET, Dapper, EF Core, LINQ-to-SQL, NHibernate, and database client wrappers
- admin panels, reporting systems, exports, dashboards, search, filter, sort, and saved query features

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real query sink in the target code.

---

# 1. C# / .NET SQL Control Points

## 1.1 Entry and API points
Look for:
- ASP.NET controllers and minimal APIs
- Razor Page handlers
- SignalR hub methods
- gRPC/WCF service methods
- Azure Functions triggers
- queue consumers and background workers
- admin, report, dashboard, and export handlers

## 1.2 Query construction and execution points
Look for:
- ADO.NET command text
- EF Core raw SQL APIs
- Dapper string queries
- dynamic LINQ and structural ordering
- stored procedure calls with dynamic SQL
- report/query helper wrappers

## 1.3 Safety controls
Look for:
- `DbParameter` usage
- `SqlParameter` usage
- EF interpolated APIs
- Dapper anonymous parameters
- strict structural allowlists
- fixed stored procedure names and parameter binding

---

# 2. High-Coverage C# / .NET SQL Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, query construction behavior, sink execution, and missing binding or structural controls.

## 2.1 ASP.NET, Web API, Razor Pages, and minimal API entry candidates
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
- `HttpRequest`
- `Request.Query`
- `Request.Form`
- `Request.Cookies`
- `MapGet`
- `MapPost`
- `MapPut`
- `MapDelete`
- `PageModel`
- `OnGet`
- `OnPost`

## 2.2 RPC, function, worker, report, and admin entries
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
- `Admin`
- `Dashboard`
- `Report`
- `Export`
- `Search`
- `Filter`
- `Sort`
- `SavedQuery`

## 2.3 Query construction and structural fragment candidates
Search for:
- `string sql`
- `string query`
- `CommandText`
- `whereClause`
- `condition`
- `filter`
- `orderBy`
- `sort`
- `sortField`
- `sortDirection`
- `column`
- `table`
- `operator`
- `limit`
- `offset`
- `StringBuilder`
- `string.Format`
- `$"{`
- `FormattableString`
- `RawSqlString`
- `FromSql`
- `FromSqlRaw`
- `ExecuteSqlRaw`
- `SqlQueryRaw`
- `customQuery`
- `reportSql`

## 2.4 ADO.NET and database client sink candidates
Search for:
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
- `SqlConnection`
- `DbConnection`
- `CreateCommand`
- `Parameters.Add`
- `Parameters.AddWithValue`
- `SqlParameter`
- `DbParameter`

## 2.5 EF Core, Dapper, NHibernate, and raw ORM sink candidates
Search for:
- `FromSqlRaw`
- `FromSqlInterpolated`
- `ExecuteSqlRaw`
- `ExecuteSqlInterpolated`
- `SqlQueryRaw`
- `SqlQueryInterpolated`
- `Database.ExecuteSqlRaw`
- `Database.SqlQueryRaw`
- `Dapper`
- `.Query<`
- `.QueryAsync`
- `.QueryFirst`
- `.QuerySingle`
- `.Execute`
- `.ExecuteAsync`
- `DynamicParameters`
- `NHibernate`
- `CreateSQLQuery`
- `CreateQuery`
- `LinqKit`
- `System.Linq.Dynamic`
- `OrderBy(string`
- `Where(string`
- `DynamicExpressionParser`

## 2.6 Stored procedure and structural SQL candidates
Search for:
- `CommandType.StoredProcedure`
- `EXEC `
- `EXECUTE `
- `sp_executesql`
- stored procedure name from variable
- `ORDER BY ` concatenation
- `WHERE ` concatenation
- `SELECT ` concatenation
- `FROM ` concatenation
- `OFFSET`
- `FETCH NEXT`
- `TOP`
- `LIKE '%`
- saved filter
- report template

## 2.7 Required-control candidates
Search near sinks for:
- `SqlParameter`
- `DbParameter`
- `Parameters.Add`
- `Parameters.AddWithValue`
- `DynamicParameters`
- anonymous Dapper parameters
- `FromSqlInterpolated`
- `ExecuteSqlInterpolated`
- `FormattableString`
- fixed `CommandText`
- `CommandType.StoredProcedure`
- `Enum`
- `switch`
- allowed columns
- allowed sorts
- allowed tables
- `Regex.IsMatch`
- `Validate`
- typed LINQ
- expression tree mapping

## 2.8 C# / .NET graph search recipes
Useful combinations:

```text
[HttpGet] + SqlCommand
[FromQuery] + CommandText
MapPost + FromSqlRaw
HttpTrigger + Dapper Query
QueueTrigger + ExecuteSqlRaw
Request.Query + string sql
OrderBy(string) + request
FromSqlRaw + interpolation
SqlCommand + string.Format
savedFilter + CommandText
sp_executesql + concatenated SQL
```

---

# 3. C# / .NET SQL Injection Anti-Patterns

### A1. Concatenated CommandText
```csharp
cmd.CommandText = "select * from users where name = '" + name + "'";
cmd.ExecuteReader();
```

Why risky:
User input is inserted directly into executable SQL text.

### A2. EF Core FromSqlRaw interpolation
```csharp
context.Users.FromSqlRaw($"select * from users where name = '{name}'");
```

Why risky:
`FromSqlRaw` executes raw SQL text and does not protect interpolated fragments.

### A3. Dapper query with dynamic ORDER BY
```csharp
connection.Query("select * from orders order by " + sort);
```

Why risky:
Structural SQL elements require strict allowlisting.

### A4. Stored procedure builds dynamic SQL
```csharp
cmd.CommandText = "exec Search '" + filter + "'";
```

Why risky:
Unsafe construction can occur before or inside stored procedure execution.

---

# 4. Case Templates

## Case C-SQL-1: ADO.NET raw query injection

### Vulnerable pattern
```csharp
cmd.CommandText = "select * from users where username = '" + username + "'";
```

### Safer pattern
```csharp
cmd.CommandText = "select * from users where username = @username";
cmd.Parameters.AddWithValue("@username", username);
```

## Case C-SQL-2: EF Core raw SQL misuse

### Vulnerable pattern
```csharp
context.Reports.FromSqlRaw("select * from report where type = '" + type + "'");
```

## Case C-SQL-3: Dapper structural injection

### Vulnerable pattern
```csharp
connection.Query("select * from products order by " + sort);
```

---

# 5. C# / .NET-Specific Audit Heuristics

## 5.1 ADO.NET heuristics
Pay attention to `CommandText`, `ExecuteReader`, `ExecuteScalar`, `ExecuteNonQuery`, and parameter collection usage.

## 5.2 EF Core heuristics
Review `FromSqlRaw`, `ExecuteSqlRaw`, and `SqlQueryRaw`; prefer interpolated APIs only for values, not structure.

## 5.3 Dapper heuristics
Review raw SQL strings and ensure values use anonymous parameters while structural pieces are allowlisted.

## 5.4 Dynamic LINQ heuristics
Review string-based `OrderBy`, `Where`, and expression parser calls for request-controlled structure.
