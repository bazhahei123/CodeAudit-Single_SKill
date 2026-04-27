# Java SQL Injection Source Cases

## Purpose

This file contains Java-specific source point patterns and audit cases for SQL injection source discovery.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- Spring JDBC
- MyBatis
- JPA / Hibernate
- jOOQ
- Servlet-based web applications
- Java backends exposing REST APIs, admin panels, reporting systems, analytics, or search functionality

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream SQL behavior in the target code.

---

# 1. Java Source Discovery Points

Prioritize these source values and events:
- `@PathVariable`, `@RequestParam`, `@RequestBody`, headers, cookies, multipart metadata, and import DTO fields
- search terms, filter keys, filter values, operators, sort fields, sort directions, page size, limit, offset, and report parameters
- table names, column names, select fields, raw conditions, query templates, dashboard fields, and tenant/schema selectors
- values passed into `JdbcTemplate`, `Statement`, `PreparedStatement`, `EntityManager`, MyBatis mappers, jOOQ builders, repository helpers, and query wrapper services
- MyBatis `${}` values and provider-based SQL builder inputs
- stored filters, report templates, saved search rules, queued job arguments, and admin replay data

Source questions:
- Which source supplies query value, filter key, operator, sort field, table, column, raw fragment, or template?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Is the value bound, allowlisted, mapped to a safe enum, formatted, concatenated, stored, or passed through a wrapper before query use?
- Which SQL context should be audited next: bound value, structural selection, raw SQL text, ORM helper, MyBatis substitution, stored procedure, or report generator?

---

# 2. Java Source Patterns

## J-S1. Request-derived query value
Example idea:
- request value such as username, keyword, status, ID, date range, or tenant selector becomes a repository or DAO query argument.

Audit relevance:
The value may be safe only if it is bound or passed through a safe typed API downstream.

Follow-up:
- trace into JDBC, Spring JDBC, JPA/Hibernate, MyBatis, jOOQ, or repository helpers.

## J-S2. Dynamic search or filter source
Example idea:
- request or DTO field selects filter key, operator, filter value, or advanced search condition.

Audit relevance:
Filter keys and operators can become structural SQL sources even when filter values are bound.

Follow-up:
- verify fixed mappings, allowed operators, and safe criteria-building APIs.

## J-S3. Sort, pagination, and structural source
Example idea:
- request value selects `sort`, `direction`, `column`, `table`, `limit`, `offset`, selected fields, or report grouping.

Audit relevance:
Prepared statements do not protect most structural choices.

Follow-up:
- verify strict allowlists, enum mappings, numeric constraints, and no raw fragment substitution.

## J-S4. Raw SQL fragment or template source
Example idea:
- formatted string, `StringBuilder`, saved condition, report template, or admin-defined query fragment is passed toward JDBC, JPA native query, or query helper.

Audit relevance:
Raw fragments are high-priority source points when the writer is weakly trusted.

Follow-up:
- inspect binding, allowlists, and whether raw clauses are replaced with structured criteria.

## J-S5. MyBatis, ORM, and query-wrapper source
Example idea:
- mapper parameters, `${}` substitutions, `@Query` fragments, `EntityManager` query text, `JdbcTemplate` SQL string, or wrapper helper arguments receive request-derived strings.

Audit relevance:
Framework abstractions can hide SQL-relevant source movement.

Follow-up:
- inspect mapper XML, annotations, provider methods, and helper implementations.

## J-S6. Stored filter or report source
Example idea:
- saved filter, dashboard definition, report template, or queued job argument later constructs a SQL query.

Audit relevance:
Stored and background values create second-order SQL source paths.

Follow-up:
- identify writer path and revalidation before query use.

---

# 3. Case Templates

## Case J-S-SQL-1: Repository query value source

Source focus:
Identify request, DTO, or stored values that become DAO/repository query arguments.

Recommended follow-up:
Verify the final data-access call uses real binding or safe typed query APIs.

## Case J-S-SQL-2: MyBatis structural source

Source focus:
Identify mapper parameters that feed `${}`, dynamic XML fragments, provider SQL builders, or raw `ORDER BY` / `WHERE` fragments.

Recommended follow-up:
Verify structural values are mapped to fixed server-side choices.

## Case J-S-SQL-3: JPA/Hibernate query text source

Source focus:
Identify request or stored values that reach JPQL/HQL/native query strings, `@Query`, or `EntityManager` query construction.

Recommended follow-up:
Verify parameter binding for values and allowlists for structure.

## Case J-S-SQL-4: Report or saved filter source

Source focus:
Identify saved filters, report definitions, dashboard metadata, or admin-configured query settings that later shape SQL.

Recommended follow-up:
Trace writer and reader paths and verify stored data is structured and revalidated.

---

# 4. Java-Specific Audit Heuristics

## 4.1 Spring route and DTO source heuristics
Pay attention to:
- `@RequestParam`
- `@PathVariable`
- `@RequestBody` DTO fields
- `@RequestHeader`
- multipart metadata
- GraphQL resolver arguments
- admin/report/search route parameters

## 4.2 Spring JDBC and raw JDBC source heuristics
Pay attention to:
- values flowing into SQL strings used by `JdbcTemplate`
- values added to `StringBuilder`, `String.format`, or concatenated SQL
- values passed to `Statement` and `PreparedStatement` setup logic
- sort/filter fragments built before data-access calls
- helper methods returning SQL strings or fragments

## 4.3 MyBatis source heuristics
Pay attention to:
- mapper method parameters
- `${}` substitutions
- dynamic XML fragments
- provider-based SQL builders
- `ORDER BY`, `WHERE`, `LIMIT`, and table/column fragments
- raw string parameters in mapper interfaces

## 4.4 JPA, Hibernate, and jOOQ source heuristics
Pay attention to:
- `EntityManager.createQuery(...)`
- `EntityManager.createNativeQuery(...)`
- string-built JPQL/HQL
- `@Query` with dynamic fragments
- Criteria API values vs raw expression strings
- jOOQ plain SQL helpers

## 4.5 Stored and second-order source heuristics
Pay attention to:
- saved search rules
- report templates
- admin-defined conditions
- dashboard column selections
- stored sort/filter metadata
- queue job arguments
- export/report background workers

---

# 5. False-Positive Controls

Do not mark a Java source as high-priority if:
- the value is selected from a strict allowlist of safe columns, tables, operators, directions, or report templates,
- the value is used only as a safely bound parameter and no structural SQL is influenced,
- the value never reaches query construction, query builder options, raw fragments, ORM helpers, or stored query metadata,
- the stored value is trusted-only and cannot be attacker-influenced.

Use `Suspected source` or `Not enough evidence` if:
- repository/helper behavior is hidden,
- mapper XML or annotation behavior is not visible,
- stored filter writer paths are missing,
- value-binding vs structural context is unclear,
- allowlist behavior may exist elsewhere.

---

# 6. What Good Evidence Looks Like

Good Java source evidence includes:
- route/controller/worker/admin/import entry point,
- source annotation or API such as request param, DTO field, mapper argument, queue receive, config record, or stored filter,
- propagation such as DTO mapping, criteria construction, SQL string assembly, MyBatis parameter passing, report template loading, or wrapper calls,
- JDBC, Spring JDBC, JPA/Hibernate, MyBatis, jOOQ, repository helper, report generator, or stored procedure path receiving the value,
- SQL context when visible.

Good source evidence answers:
1. Which Java entry point receives the SQL-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which value-binding, structural, raw fragment, ORM, mapper, report, or worker behavior should be audited next?
4. Is the source used for query value, filter key, operator, sort field, column, table, raw condition, template, or stored query metadata?

---

# 7. Quick Java Source Checklist

- Are request values used as query values, filter keys, operators, sort fields, or pagination controls?
- Are table names, columns, selected fields, report fields, or schemas dynamic?
- Are raw strings passed into JDBC, `JdbcTemplate`, `EntityManager`, MyBatis, jOOQ, or repository wrappers?
- Are MyBatis `${}` values externally influenced?
- Are saved filters, report definitions, or dashboard metadata later used to build SQL?
- Are export/report/admin query sources handled the same way as primary user routes?
