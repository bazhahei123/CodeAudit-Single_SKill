# C++ SQL Injection Source Cases

## Purpose

This file contains C++-specific source point patterns and candidate search terms for SQL injection source discovery.

Use it when the target application includes C++ or native service code, especially in:
- HTTP, RPC, WebSocket, IPC, socket, or message-bus handlers
- SQLite, MySQL/MariaDB, PostgreSQL/libpq, ODBC, SQL Server, Oracle, SOCI, Qt SQL, Poco Data, libpqxx, and custom database wrappers
- admin panels, reports, exports, dashboards, search, filter, sort, analytics, saved query, and native data-service features

This reference is guidance, not proof. C++ database code often hides SQL construction behind wrappers. Always verify source origin, propagation, trust boundary, downstream SQL relevance, and later binding or structural controls.

---

# 1. High-Coverage C++ SQL Source Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; keep it only when code evidence shows that the value can influence SQL values, SQL structure, raw fragments, database wrapper input, stored query metadata, or native DB execution behavior.

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
- `onClose`
- `recv`
- `read`
- `boost::asio`
- `DBus`
- `sd_bus_message`
- `Binder`
- `onTransact`
- custom IPC handlers

## 1.2 Message, report, export, CLI, and admin entries

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
- `admin`
- `dashboard`
- `analytics`
- `report`
- `export`
- `search`
- `filter`
- `sort`
- `savedFilter`
- `savedQuery`
- `reportTemplate`
- `runReport`
- `queryService`
- import/sync workers
- replay handlers

## 1.3 SQL value and criteria source candidates

Search for request, message, protobuf, JSON, CLI, config, or DTO fields named:
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
- `tenant_id`
- `accountId`
- `userId`
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
- `std::string sql`
- `std::string query`
- `std::stringstream`
- `std::ostringstream`
- `fmt::format`
- `absl::StrFormat`
- `boost::format`
- `QString`
- `QString::arg`
- concatenation around `WHERE`, `ORDER BY`, `LIMIT`, `OFFSET`, `SELECT`, `FROM`, or `JOIN`

## 1.6 Native DB, ORM, and wrapper input candidates

Search for source values passed into:
- custom `Database::execute` wrappers
- custom `executeQuery` wrappers
- SQLite query helpers
- MySQL query helpers
- libpq query helpers
- ODBC query helpers
- Qt SQL helpers
- SOCI session wrappers
- Poco Data statement wrappers
- libpqxx transaction wrappers
- `QSqlQuery`
- `soci::session`
- `Poco::Data::Session`
- `pqxx::work`
- `Statement`
- `query builder`
- report/query/export services
- saved filter readers and query metadata loaders

## 1.7 Downstream SQL relevance mapping candidates

After finding a source candidate, trace toward:
- `sqlite3_exec`
- `sqlite3_prepare`
- `sqlite3_prepare_v2`
- `sqlite3_prepare_v3`
- `sqlite3_bind_text`
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
- `QSqlQuery::exec`
- `QSqlQuery::prepare`
- `QSqlQuery::bindValue`
- `soci::use`
- `exec_params`
- repository, report, export, analytics, or dashboard query helpers

## 1.8 C++ graph search recipes

Useful combinations:

```text
CROW_ROUTE/svr.Get/ENDPOINT + query/filter/sort + database wrapper
grpc::Service/ServerContext + whereClause/orderBy + PQexec/mysql_query/sqlite3_exec
recv/onMessage/IPC handler + rawSql/customQuery + executeQuery
CLI argv/getopt + reportTemplate/savedQuery + query builder/helper
std::stringstream/fmt::format/QString::arg + request/stored field + WHERE/ORDER BY/LIMIT
QSqlQuery/SOCI/libpqxx wrapper + dynamic column/table/operator source
```

---

# 2. C++ Source Patterns

## CPP-S1. Network, RPC, IPC, or CLI source becomes query criteria

Example idea:
- HTTP query parameter, protobuf field, socket message, IPC payload, or CLI argument becomes a database wrapper argument.

Audit relevance:
Native service boundaries are often custom; source classification should identify the protocol, parser, and first query-relevant wrapper call.

Follow-up:
- trace into SQLite, MySQL, PostgreSQL/libpq, ODBC, Qt SQL, SOCI, Poco Data, libpqxx, or custom wrappers.

## CPP-S2. Dynamic structural selector source

Example idea:
- request or stored value selects sort field, direction, column, table, grouping, limit, or procedure name.

Audit relevance:
Prepared statements do not protect most structural SQL choices.

Follow-up:
- verify fixed enum mappings, allowlists, and safe query builder DSL behavior.

## CPP-S3. Raw SQL text or template source

Example idea:
- `std::string`, `stringstream`, `fmt::format`, `QString::arg`, stored filter, or report template creates SQL text from weakly trusted values.

Audit relevance:
String-built SQL is common in C++ wrappers and should be classified as a source when attacker influence is possible.

Follow-up:
- inspect wrapper internals, parameter APIs, and whether structural fragments are fixed.

## CPP-S4. Stored, native service, or report source

Example idea:
- saved query metadata, imported records, sync data, report templates, or queue messages later shape SQL in native workers.

Audit relevance:
Stored and background values create second-order SQL source paths.

Follow-up:
- identify writer path and revalidation before SQL construction.

---

# 3. False-Positive Controls

Do not mark a C++ source as high-priority if:
- the value is fixed in trusted code or selected from strict enum/allowlist mappings,
- the value is bound only as a prepared-statement value with no structural influence,
- wrapper internals prove safe binding and fixed SQL structure,
- the candidate source never reaches SQL construction, database wrappers, or query metadata.

Use `Suspected source` or `Not enough evidence` if:
- protocol parsing or caller trust is unclear,
- custom database wrappers hide SQL construction,
- string construction is visible but execution behavior is not,
- stored filter writer paths are missing.

---

# 4. Quick C++ Source Checklist

- Do HTTP/RPC/WebSocket/IPC/socket/message/CLI inputs become SQL criteria or query options?
- Do request or stored values influence raw SQL strings, table names, column names, sort/order fields, operators, limit/offset, or report templates?
- Are C++ string builders, formatters, `QString::arg`, or custom wrappers carrying weakly trusted values toward database APIs?
- Are SQLite, MySQL, libpq, ODBC, Qt SQL, SOCI, Poco Data, or libpqxx calls reached by external or stored values?
- Is the source only a bound value, or can it change SQL structure?
