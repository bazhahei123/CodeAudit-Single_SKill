# Python SQL Injection Source Cases

## Purpose

This file contains Python-specific source point patterns and audit cases for SQL injection source discovery.

Use it when the target application is primarily implemented in Python, especially in:
- Django
- Django REST Framework
- Flask
- FastAPI
- Starlette
- SQLAlchemy-based services
- raw DB-API code
- Python backends exposing REST APIs, admin panels, reporting systems, analytics, or search functionality

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream SQL behavior in the target code.

---

# 1. Python Source Discovery Points

Prioritize these source values and events:
- route parameters, query values, form fields, JSON body fields, headers, cookies, uploaded metadata, and import rows
- search terms, filter keys, filter values, operators, sort fields, sort directions, page size, limit, offset, and report parameters
- table names, column names, select fields, raw conditions, SQL text snippets, report templates, dashboard fields, and tenant/schema selectors
- values passed into `cursor.execute(...)`, `executemany(...)`, Django `.raw(...)`, `connection.cursor()`, SQLAlchemy `text(...)`, `.execute(...)`, query helpers, repository wrappers, and report generators
- stored filters, report templates, saved search rules, Celery/task payloads, queued job arguments, and admin replay data

Source questions:
- Which source supplies query value, filter key, operator, sort field, table, column, raw fragment, or template?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value bound, allowlisted, mapped to a safe enum, formatted, concatenated, stored, or passed through a wrapper before query use?
- Which SQL context should be audited next: bound value, structural selection, raw SQL text, ORM helper, SQLAlchemy text, Django raw query, stored procedure, or report generator?

---

# 2. Python Source Patterns

## P-S1. Request-derived query value
Example idea:
- request value such as username, keyword, status, ID, date range, or tenant selector becomes a query helper or ORM argument.

Audit relevance:
The value may be safe only if it is bound or passed through a safe typed API downstream.

Follow-up:
- trace into DB-API, Django ORM/raw queries, SQLAlchemy, repository helpers, or report generators.

## P-S2. Dynamic search or filter source
Example idea:
- request JSON, query args, or saved filter fields select filter key, operator, or advanced search condition.

Audit relevance:
Filter keys and operators can become structural SQL sources even when filter values are bound.

Follow-up:
- verify fixed mappings, allowed operators, and safe criteria-building APIs.

## P-S3. Sort, pagination, and structural source
Example idea:
- request value selects `sort`, `direction`, `column`, `table`, `limit`, `offset`, selected fields, or grouping.

Audit relevance:
Driver placeholders do not protect most structural choices.

Follow-up:
- verify strict allowlists, enum mappings, numeric constraints, and no raw fragment substitution.

## P-S4. Raw SQL fragment or template source
Example idea:
- f-string input, `.format(...)` argument, stored condition, report template, or admin-defined query fragment is passed toward cursor execution, Django `.raw(...)`, or SQLAlchemy `text(...)`.

Audit relevance:
Raw fragments are high-priority source points when the writer is weakly trusted.

Follow-up:
- inspect binding, allowlists, and whether raw clauses are replaced with structured criteria.

## P-S5. ORM and query-wrapper source
Example idea:
- Django `.raw(...)`, `extra(...)`, SQLAlchemy `text(...)`, `literal_column(...)`, dynamic `order_by`, or repository wrapper receives request-derived strings.

Audit relevance:
Framework abstractions can hide SQL-relevant source movement.

Follow-up:
- inspect wrapper internals and SQLAlchemy/Django argument semantics.

## P-S6. Stored filter or worker source
Example idea:
- saved filter, dashboard definition, report template, Celery task argument, or queued job payload later constructs a SQL query.

Audit relevance:
Stored and background values create second-order SQL source paths.

Follow-up:
- identify writer path and revalidation before query use.

---

# 3. Case Templates

## Case P-S-SQL-1: DB-API query value source

Source focus:
Identify request or stored values that become arguments to cursor execution or helper methods around DB-API calls.

Recommended follow-up:
Verify true driver binding and no mixed unsafe structural fragments.

## Case P-S-SQL-2: Django raw query source

Source focus:
Identify request, DTO, or stored values that reach Django `.raw(...)`, `connection.cursor()`, custom managers, or raw report helpers.

Recommended follow-up:
Verify values are bound and structural choices are fixed or allowlisted.

## Case P-S-SQL-3: SQLAlchemy text or raw fragment source

Source focus:
Identify sources passed into `text(...)`, `.execute(...)`, `literal_column(...)`, dynamic `order_by`, or plain SQL wrappers.

Recommended follow-up:
Verify bind parameters for values and allowlists for structural inputs.

## Case P-S-SQL-4: Worker or saved filter source

Source focus:
Identify saved filters, report definitions, dashboard metadata, Celery task payloads, or queued query options that later shape SQL.

Recommended follow-up:
Trace writer and reader paths and verify stored data is structured and revalidated.

---

# 4. Python-Specific Audit Heuristics

## 4.1 Django and DRF source heuristics
Pay attention to:
- `request.GET`
- `request.POST`
- `request.data`
- route kwargs
- serializer fields used in query helpers
- `.raw(...)`
- `connection.cursor()`
- report/export endpoints using manual SQL

## 4.2 Flask and FastAPI source heuristics
Pay attention to:
- `request.args`
- `request.form`
- `request.get_json()`
- Pydantic model fields
- dependency-injected query options
- service-layer SQL builder functions
- f-string or `.format(...)` query helpers

## 4.3 SQLAlchemy source heuristics
Pay attention to:
- `text(...)`
- `.execute(...)`
- dynamic `order_by`
- `literal_column(...)`
- `column(...)` with external names
- mixed safe and raw statements
- helper methods that accept string fragments

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
- Celery task arguments
- queued report jobs
- import rows later used in query generation

---

# 5. False-Positive Controls

Do not mark a Python source as high-priority if:
- the value is selected from a strict allowlist of safe columns, tables, operators, directions, or report templates,
- the value is used only as a safely bound parameter and no structural SQL is influenced,
- the value never reaches query construction, query builder options, raw fragments, ORM helpers, or stored query metadata,
- the stored value is trusted-only and cannot be attacker-influenced.

Use `Suspected source` or `Not enough evidence` if:
- repository/helper behavior is hidden,
- Django or SQLAlchemy argument semantics are unclear,
- stored filter writer paths are missing,
- value-binding vs structural context is unclear,
- allowlist behavior may exist elsewhere.

---

# 6. What Good Evidence Looks Like

Good Python source evidence includes:
- route/view/worker/admin/import entry point,
- source API such as request args, JSON body, form value, uploaded metadata, queue payload, config record, or stored filter,
- propagation such as schema/model mapping, query option assembly, f-string formatting, SQLAlchemy expression construction, report template loading, storage, or wrapper call,
- DB-API cursor, Django raw query, SQLAlchemy execution path, repository helper, report generator, or stored procedure path receiving the value,
- SQL context when visible.

Good source evidence answers:
1. Which Python entry point receives the SQL-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which value-binding, structural, raw fragment, ORM, report, or worker behavior should be audited next?
4. Is the source used for query value, filter key, operator, sort field, column, table, raw condition, template, or stored query metadata?

---

# 7. Quick Python Source Checklist

- Are request values used as query values, filter keys, operators, sort fields, or pagination controls?
- Are table names, columns, selected fields, report fields, or schemas dynamic?
- Are raw strings passed into DB-API cursors, Django raw queries, SQLAlchemy `text(...)`, or repository wrappers?
- Are saved filters, report definitions, or dashboard metadata later used to build SQL?
- Are export/report/admin query sources handled the same way as primary user routes?
- Are worker and queued report sources revalidated before query use?
