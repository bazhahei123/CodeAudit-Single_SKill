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

# 2. PHP Source Patterns

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

# 3. Case Templates

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

# 4. PHP-Specific Audit Heuristics

## 4.1 Laravel source heuristics
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

## 4.2 PDO and mysqli source heuristics
Pay attention to:
- `$_GET`, `$_POST`, and route values flowing into SQL strings
- values interpolated before `prepare(...)`
- partial placeholder usage
- manual escaping paths
- helper methods constructing SQL strings centrally
- raw admin/report scripts

## 4.3 Symfony, ThinkPHP, Yii, and CodeIgniter source heuristics
Pay attention to:
- raw SQL through DBAL connection objects
- `executeQuery(...)`
- query builder raw fragments
- dynamic order or filter pieces
- repository helpers returning raw SQL
- framework-specific raw expression helpers

## 4.4 Structural control source heuristics
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

## 4.5 Stored and second-order source heuristics
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

# 5. False-Positive Controls

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

# 6. What Good Evidence Looks Like

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

# 7. Quick PHP Source Checklist

- Are request values used as query values, filter keys, operators, sort fields, or pagination controls?
- Are table names, columns, selected fields, report fields, or schemas dynamic?
- Are raw strings passed into PDO, mysqli, Laravel raw helpers, Symfony DBAL, or repository wrappers?
- Are saved filters, report definitions, or dashboard metadata later used to build SQL?
- Are export/report/admin query sources handled the same way as primary user routes?
- Are worker, cron, and queued report sources revalidated before query use?
