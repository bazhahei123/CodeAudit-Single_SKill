# C++ SQL Injection Cases

## Purpose

This file contains C++-specific SQL injection patterns, candidate sink inventories, and audit cases.

Use it when the target application includes C++ or native service code, especially in:
- HTTP, RPC, WebSocket, IPC, socket, or message-bus handlers
- SQLite, MySQL/MariaDB, PostgreSQL/libpq, ODBC, SQL Server, Oracle, SOCI, Qt SQL, Poco Data, and custom database wrappers
- admin panels, reports, exports, dashboards, search, filter, sort, analytics, and saved query features

This reference is guidance, not proof. C++ database code often hides SQL construction behind wrappers. Always verify attacker influence, query construction behavior, sink execution, and missing binding or structural controls.

---

# 1. C++ SQL Control Points

## 1.1 Entry and task points
Look for:
- HTTP route handlers
- RPC or gRPC service methods
- socket, pipe, DBus, Binder, or custom IPC handlers
- message queue consumers
- admin, report, export, and search paths
- native database wrapper services

## 1.2 Query construction and execution points
Look for:
- string-built SQL
- database client execution APIs
- prepared statement APIs
- ORM or query builder raw fragments
- dynamic report and saved filter systems

## 1.3 Safety controls
Look for:
- prepared statements
- bound parameters
- fixed server-side mappings for structural elements
- strict allowlists for table, column, sort, and operator choices

---

# 2. High-Coverage C++ SQL Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, query construction behavior, sink execution, and missing binding or structural controls.

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
- `DBus`
- `sd_bus_message`
- `Binder`
- `onTransact`

## 2.2 Message, report, export, and admin entries
Search for:
- `KafkaConsumer`
- `RdKafka`
- `AMQP`
- `RabbitMQ`
- `MQTT`
- `ZeroMQ`
- `boost::asio`
- `admin`
- `dashboard`
- `analytics`
- `report`
- `export`
- `search`
- `filter`
- `sort`
- `savedFilter`
- `reportTemplate`
- `runReport`
- `queryService`

## 2.3 Query construction and structural fragment candidates
Search for:
- `std::string sql`
- `std::string query`
- `std::stringstream`
- `fmt::format`
- `absl::StrFormat`
- `QString`
- `QString::arg`
- `boost::format`
- `append`
- `operator+`
- `WHERE`
- `ORDER BY`
- `LIMIT`
- `OFFSET`
- `whereClause`
- `condition`
- `filter`
- `orderBy`
- `sort`
- `column`
- `table`
- `operator`
- `customQuery`
- `reportSql`

## 2.4 SQLite, MySQL, PostgreSQL, ODBC, and native DB sink candidates
Search for:
- `sqlite3_exec`
- `sqlite3_prepare`
- `sqlite3_prepare_v2`
- `sqlite3_prepare_v3`
- `sqlite3_bind_text`
- `sqlite3_bind_int`
- `mysql_query`
- `mysql_real_query`
- `mysql_stmt_prepare`
- `mysql_stmt_bind_param`
- `PQexec`
- `PQexecParams`
- `PQprepare`
- `PQexecPrepared`
- `SQLExecDirect`
- `SQLExecute`
- `SQLPrepare`
- `SQLBindParameter`
- `OCIStmtExecute`
- `SQLCommand`
- `Database::execute`
- `executeQuery`
- `runQuery`

## 2.5 Framework, ORM, and wrapper sink candidates
Search for:
- `QtSql`
- `QSqlQuery`
- `QSqlQuery::exec`
- `QSqlQuery::prepare`
- `QSqlQuery::bindValue`
- `QSqlDatabase`
- `SOCI`
- `soci::session`
- `soci::use`
- `soci::into`
- `Poco::Data`
- `Poco::Data::Session`
- `Statement`
- `ODB`
- `Wt::Dbo`
- `sqlpp11`
- `sqlite_orm`
- `libpqxx`
- `pqxx::work`
- `pqxx::nontransaction`
- `exec_params`
- `exec_prepared`

## 2.6 Required-control candidates
Search near sinks for:
- prepared statement
- `sqlite3_bind`
- `mysql_stmt_bind_param`
- `PQexecParams`
- `PQexecPrepared`
- `SQLBindParameter`
- `QSqlQuery::prepare`
- `QSqlQuery::bindValue`
- `soci::use`
- `exec_params`
- `parameter`
- `placeholder`
- `allowlist`
- `allowedColumns`
- `allowedSorts`
- `allowedTables`
- `enum`
- `switch`
- `std::regex_match`
- `validate`
- fixed query templates
- safe query builder DSL

## 2.7 C++ graph search recipes
Useful combinations:

```text
CROW_ROUTE + sqlite3_exec
svr.Get + mysql_query
grpc::Service + PQexec
recv + SQLExecDirect
QSqlQuery::exec + QString::arg
std::stringstream + sqlite3_prepare_v2
ORDER BY + request parameter + execute
savedFilter + whereClause + PQexec
reportSql + mysql_real_query
PQexec without PQexecParams
```

---

# 3. C++ SQL Injection Anti-Patterns

### A1. sqlite3_exec with concatenated SQL
```cpp
std::string sql = "select * from users where name = '" + name + "'";
sqlite3_exec(db, sql.c_str(), nullptr, nullptr, nullptr);
```

Why risky:
User input is inserted directly into executable SQL text.

### A2. MySQL raw query with dynamic ORDER BY
```cpp
std::string sql = "select * from orders order by " + sort;
mysql_query(conn, sql.c_str());
```

Why risky:
Structural SQL requires strict allowlisting.

### A3. Qt SQL string formatting
```cpp
query.exec(QString("select * from item where id = %1").arg(id));
```

Why risky:
Formatted query strings are executable SQL text.

### A4. PostgreSQL PQexec with concatenated filter
```cpp
PQexec(conn, ("select * from logs where " + whereClause).c_str());
```

Why risky:
Raw condition fragments can give attackers direct SQL logic control.

---

# 4. Case Templates

## Case CPP-SQL-1: SQLite raw query injection

### Vulnerable pattern
```cpp
sqlite3_exec(db, ("select * from users where name = '" + name + "'").c_str(), nullptr, nullptr, nullptr);
```

### Safer pattern
```cpp
sqlite3_prepare_v2(db, "select * from users where name = ?", -1, &stmt, nullptr);
sqlite3_bind_text(stmt, 1, name.c_str(), -1, SQLITE_TRANSIENT);
```

## Case CPP-SQL-2: PostgreSQL raw query injection

### Vulnerable pattern
```cpp
PQexec(conn, sql.c_str());
```

### Audit focus
Verify whether `PQexecParams` or prepared execution is used for values and allowlists are used for structure.

## Case CPP-SQL-3: Qt dynamic query injection

### Vulnerable pattern
```cpp
QSqlQuery q;
q.exec("select * from product order by " + sort);
```

---

# 5. C++-Specific Audit Heuristics

## 5.1 SQLite heuristics
Review `sqlite3_exec`, `sqlite3_prepare*`, and bind usage; raw `exec` is high priority when SQL is dynamic.

## 5.2 MySQL/PostgreSQL heuristics
Prefer `mysql_stmt_*` and `PQexecParams`/prepared queries over raw `mysql_query` and `PQexec`.

## 5.3 ODBC and enterprise DB heuristics
Review `SQLExecDirect`, `SQLPrepare`, `SQLBindParameter`, and wrapper methods that hide raw SQL strings.

## 5.4 Qt/SOCI/Poco heuristics
Review `QSqlQuery::exec`, SOCI statement construction, Poco Data statements, and any custom query helper accepting strings.
