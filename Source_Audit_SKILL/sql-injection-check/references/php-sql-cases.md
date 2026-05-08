# PHP SQL Injection Source Cases

## Purpose

This file contains PHP-specific source point patterns and audit cases for SQL injection source discovery.

Use it when the target application is primarily implemented in PHP, especially in:
- Laravel
- Symfony
- ThinkPHP
- Yii
- CodeIgniter
- raw PDO / mysqli code
- PHP backends exposing REST APIs, admin panels, reporting systems, analytics, or search functionality

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream SQL behavior in the target code.

---

# 1. PHP Source Discovery Points

Prioritize these source values and events:
- `$_GET`, `$_POST`, `$_REQUEST`, route parameters, request input, headers, cookies, uploaded metadata, and import rows
- search terms, filter keys, filter values, operators, sort fields, sort directions, page size, limit, offset, and report parameters
- table names, column names, select fields, raw conditions, SQL text snippets, report templates, dashboard fields, and tenant/schema selectors
- values passed into PDO, mysqli, Laravel `DB::select`, `DB::statement`, `DB::raw`, `whereRaw`, `orderByRaw`, Symfony DBAL calls, repository helpers, and report generators
- stored filters, report templates, saved search rules, queue/job payloads, cron/admin task definitions, and replay data

Source questions:
- Which source supplies query value, filter key, operator, sort field, table, column, raw fragment, or template?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value bound, allowlisted, mapped to a safe enum, concatenated, interpolated, stored, or passed through a wrapper before query use?
- Which SQL context should be audited next: bound value, structural selection, raw SQL text, Laravel raw helper, PDO/mysqli query, DBAL query, stored procedure, or report generator?

---

# 2. High-Coverage PHP SQL Source Candidate Inventory

Use these candidate lists to seed graph queries and text searches. Keep a candidate only when code shows SQL value construction, structural SQL selection, raw fragment creation, ORM/query-builder input, stored query metadata, or data-access wrapper relevance.

## 2.1 Web, framework, and request entry candidates

Search for:
- Laravel `Route::get`
- Laravel `Route::post`
- Laravel `Route::put`
- Laravel `Route::patch`
- Laravel `Route::delete`
- Laravel controller methods
- `$request->input`
- `$request->query`
- `$request->post`
- `$request->all`
- `$request->only`
- `$request->validated`
- `$request->header`
- route parameters
- Symfony `#[Route]`
- Symfony `Request`
- `$request->query->get`
- `$request->request->get`
- `$request->headers->get`
- ThinkPHP controllers
- Yii controllers
- CodeIgniter controllers
- WordPress `add_action`
- WordPress REST routes
- raw `$_GET`
- `$_POST`
- `$_REQUEST`
- `$_COOKIE`
- `$_SERVER`
- uploaded file metadata

## 2.2 Queue, command, admin, report, and import entries

Search for:
- Laravel jobs
- Laravel listeners
- Laravel events
- Laravel queued jobs
- Laravel console commands
- Symfony commands
- Symfony Messenger handlers
- cron scripts
- webhook controllers
- import controllers
- export controllers
- report controllers
- dashboard controllers
- admin actions
- saved search handlers
- replay handlers
- ETL/sync tasks
- legacy admin scripts

## 2.3 SQL value and criteria source candidates

Search for request, DTO, array, model, or form fields named:
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
- `tenant_id`
- `account_id`
- `user_id`
- `org_id`
- `ids`
- `from_date`
- `to_date`
- `start_date`
- `end_date`
- `filters`
- `criteria`
- `conditions`
- `rules`
- `predicate`
- `where`
- `scope`

## 2.4 Structural SQL selector source candidates

Search for:
- `sort`
- `sort_by`
- `order_by`
- `orderBy`
- `direction`
- `field`
- `fields`
- `column`
- `columns`
- `select`
- `projection`
- `table`
- `table_name`
- `schema`
- `database`
- `partition`
- `operator`
- `op`
- `comparator`
- `join`
- `group_by`
- `groupBy`
- `having`
- `limit`
- `offset`
- `page`
- `page_size`
- `cursor`
- `procedure`
- `procedure_name`

## 2.5 Raw fragment and query template source candidates

Search for:
- `sql`
- `raw_sql`
- `query_sql`
- `native_query`
- `custom_query`
- `report_sql`
- `where_clause`
- `order_clause`
- `having_clause`
- `join_clause`
- `condition`
- `expression`
- `filter_expression`
- `query_template`
- `report_template`
- `saved_query`
- `dashboard_query`
- interpolation inside SQL strings
- concatenation around `WHERE`, `ORDER BY`, `LIMIT`, `OFFSET`, `SELECT`, `FROM`, or `JOIN`
- `sprintf`
- `vsprintf`
- `implode`

## 2.6 ORM, DBAL, and query-wrapper input candidates

Search for source values passed into:
- PDO wrapper helpers
- mysqli wrapper helpers
- Laravel `DB::raw`
- Laravel `whereRaw`
- Laravel `orWhereRaw`
- Laravel `havingRaw`
- Laravel `orderByRaw`
- Laravel `selectRaw`
- Laravel `fromRaw`
- Laravel `joinRaw`
- Laravel `DB::select`
- Laravel `DB::statement`
- Eloquent scopes that accept raw fragments or dynamic columns
- Symfony DBAL `executeQuery`
- Doctrine native SQL helpers
- Doctrine DQL string builders
- ThinkPHP raw query helpers
- Yii raw SQL helpers
- CodeIgniter raw query helpers
- WordPress `$wpdb->query`
- WordPress `$wpdb->get_results`
- report/query/export services
- saved filter readers and query metadata loaders

## 2.7 Downstream SQL relevance mapping candidates

After finding a source candidate, trace toward:
- `PDO::query`
- `PDO::prepare`
- `PDOStatement::execute`
- `mysqli_query`
- `mysqli::query`
- `mysqli_prepare`
- `DB::select`
- `DB::statement`
- `DB::unprepared`
- `DB::raw`
- `whereRaw`
- `orderByRaw`
- `selectRaw`
- Symfony DBAL `executeQuery`
- Doctrine `createQuery`
- Doctrine `createNativeQuery`
- stored procedure callers
- repository, report, export, analytics, or dashboard query helpers

## 2.8 PHP graph search recipes

Useful combinations:

```text
Route::get/Route::post + $request->input filter/sort + repository/query helper
$request->query/$_GET + order_by/field + orderByRaw/DB::raw
Symfony #[Route] + where_clause/expression + DBAL executeQuery
Laravel job/command + saved_filter/report_template + query builder/helper
validated input + dynamic column/table/operator + Eloquent scope
sprintf/implode/concat + request/stored field + WHERE/ORDER BY/LIMIT
$wpdb/DB::select/PDO::query + raw_sql/custom_query/report_sql source
```

---

# 3. PHP Source Patterns

## H-S1. Request-derived query value
Example idea:
- request value such as username, keyword, status, ID, date range, or tenant selector becomes a repository, model, PDO, mysqli, or query builder argument.

Audit relevance:
The value may be safe only if it is bound or passed through a safe typed API downstream.

Follow-up:
- trace into PDO, mysqli, Laravel query builder, Eloquent, Symfony DBAL, repository helpers, or report generators.

## H-S2. Dynamic search or filter source
Example idea:
- request input or saved filter fields select filter key, operator, filter value, or advanced search condition.

Audit relevance:
Filter keys and operators can become structural SQL sources even when filter values are bound.

Follow-up:
- verify fixed mappings, allowed operators, and safe criteria-building APIs.

## H-S3. Sort, pagination, and structural source
Example idea:
- request value selects `sort`, `direction`, `column`, `table`, `limit`, `offset`, selected fields, or grouping.

Audit relevance:
Prepared statements do not protect most structural choices.

Follow-up:
- verify strict allowlists, enum mappings, numeric constraints, and no raw fragment substitution.

## H-S4. Raw SQL fragment or template source
Example idea:
- interpolated string, concatenated fragment, stored condition, report template, or admin-defined query fragment is passed toward PDO, mysqli, Laravel raw helpers, DBAL, or report SQL generation.

Audit relevance:
Raw fragments are high-priority source points when the writer is weakly trusted.

Follow-up:
- inspect binding, allowlists, and whether raw clauses are replaced with structured criteria.

## H-S5. ORM, DBAL, and query-wrapper source
Example idea:
- Laravel `DB::raw`, `whereRaw`, `orderByRaw`, raw Eloquent scopes, Symfony DBAL raw SQL, or repository wrapper receives request-derived strings.

Audit relevance:
Framework abstractions can hide SQL-relevant source movement.

Follow-up:
- inspect wrapper internals and framework API argument semantics.

## H-S6. Stored filter or background job source
Example idea:
- saved filter, dashboard definition, report template, queue payload, cron metadata, or replay data later constructs a SQL query.

Audit relevance:
Stored and background values create second-order SQL source paths.

Follow-up:
- identify writer path and revalidation before query use.

---

# 4. Case Templates

## Case H-S-SQL-1: PDO or mysqli query value source

Source focus:
Identify request or stored values that become arguments to PDO/mysqli execution or helper methods around those calls.

Recommended follow-up:
Verify true prepared execution and no mixed unsafe structural fragments.

## Case H-S-SQL-2: Laravel raw helper source

Source focus:
Identify request, model, or stored values that reach `DB::raw`, `whereRaw`, `orderByRaw`, `DB::select`, `DB::statement`, or raw Eloquent scopes.

Recommended follow-up:
Verify values are bound and structural choices are fixed or allowlisted.

## Case H-S-SQL-3: Symfony or DBAL raw fragment source

Source focus:
Identify sources passed into DBAL raw SQL, dynamic order/filter helpers, repository string fragments, or native query wrappers.

Recommended follow-up:
Verify bind parameters for values and allowlists for structural inputs.

## Case H-S-SQL-4: Background or saved filter source

Source focus:
Identify saved filters, report definitions, dashboard metadata, cron/admin task payloads, or queued query options that later shape SQL.

Recommended follow-up:
Trace writer and reader paths and verify stored data is structured and revalidated.

---

# 5. PHP-Specific Audit Heuristics

## 5.1 Laravel source heuristics
Pay attention to:
- `$request->input(...)`
- route parameters
- validation result arrays later used in query helpers
- `DB::select(...)`
- `DB::statement(...)`
- `DB::raw(...)`
- `whereRaw(...)`
- `orderByRaw(...)`
- raw Eloquent scopes or repository helpers

## 5.2 PDO and mysqli source heuristics
Pay attention to:
- `$_GET`, `$_POST`, and route values flowing into SQL strings
- values interpolated before `prepare(...)`
- partial placeholder usage
- manual escaping paths
- helper methods constructing SQL strings centrally
- raw admin/report scripts

## 5.3 Symfony, ThinkPHP, Yii, and CodeIgniter source heuristics
Pay attention to:
- raw SQL through DBAL connection objects
- `executeQuery(...)`
- query builder raw fragments
- dynamic order or filter pieces
- repository helpers returning raw SQL
- framework-specific raw expression helpers

## 5.4 Structural control source heuristics
Check whether the following are user-controlled:
- sortable field
- sort direction
- table name
- column name
- operator
- filter clause
- limit / offset expression
- select field list
- union-capable fragments

## 5.5 Stored and second-order source heuristics
Pay attention to:
- saved search rules
- report templates
- admin-defined conditions
- dashboard column selections
- stored sort/filter metadata
- queue job arguments
- cron/admin report jobs
- import rows later used in query generation

---

# 6. False-Positive Controls

Do not mark a PHP source as high-priority if:
- the value is selected from a strict allowlist of safe columns, tables, operators, directions, or report templates,
- the value is used only as a safely bound parameter and no structural SQL is influenced,
- the value never reaches query construction, query builder options, raw fragments, ORM helpers, or stored query metadata,
- the stored value is trusted-only and cannot be attacker-influenced.

Use `Suspected source` or `Not enough evidence` if:
- repository/helper behavior is hidden,
- framework raw-helper argument semantics are unclear,
- stored filter writer paths are missing,
- value-binding vs structural context is unclear,
- allowlist behavior may exist elsewhere.

---

# 7. What Good Evidence Looks Like

Good PHP source evidence includes:
- route/controller/script/worker/admin/import entry point,
- source API such as `$_GET`, request input, uploaded metadata, queue payload, config record, or stored filter,
- propagation such as request validation, query option assembly, string interpolation, raw helper argument construction, report template loading, storage, or wrapper call,
- PDO, mysqli, Laravel, Symfony DBAL, ThinkPHP/Yii/CodeIgniter query helper, repository helper, report generator, or stored procedure path receiving the value,
- SQL context when visible.

Good source evidence answers:
1. Which PHP entry point receives the SQL-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which value-binding, structural, raw fragment, ORM, report, or worker behavior should be audited next?
4. Is the source used for query value, filter key, operator, sort field, column, table, raw condition, template, or stored query metadata?

---

# 8. Quick PHP Source Checklist

- Are request values used as query values, filter keys, operators, sort fields, or pagination controls?
- Are table names, columns, selected fields, report fields, or schemas dynamic?
- Are raw strings passed into PDO, mysqli, Laravel raw helpers, Symfony DBAL, or repository wrappers?
- Are saved filters, report definitions, or dashboard metadata later used to build SQL?
- Are export/report/admin query sources handled the same way as primary user routes?
- Are worker, cron, and queued report sources revalidated before query use?
