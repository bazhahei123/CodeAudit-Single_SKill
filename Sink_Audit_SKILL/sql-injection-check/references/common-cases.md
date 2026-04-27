# SQL Injection Common Cases

## Purpose

This file defines shared SQL injection concepts, audit logic, anti-patterns, false-positive controls, and finding standards that apply across languages and frameworks.

Use this file as the base reference for SQL injection review before loading any language-specific reference.

This file explains:
- what SQL injection is,
- what counts as a source, sink, and control,
- how to distinguish value injection from structural injection,
- how to reason about second-order SQL injection,
- when to report `Confirmed`, `Suspected`, or `Not enough evidence`.

This reference is guidance, not proof. Do not report a vulnerability only because code resembles a pattern described here. Always verify the real data flow and real query sink in the target code.

---

# 1. Core Concepts

## 1.1 What SQL injection is
SQL injection occurs when untrusted input can influence the structure, logic, or behavior of a SQL query.

This can happen when user-controlled data is inserted into:
- string-literal positions,
- numeric positions,
- query operators,
- WHERE fragments,
- ORDER BY expressions,
- LIMIT / OFFSET expressions,
- table names,
- column names,
- UNION-capable fragments,
- stored query templates or saved filters later reused in SQL.

The core question is:

**Can attacker-controlled input change what the database executes, rather than only supplying a safe value?**

## 1.2 Source, propagation, and sink

### Source
A source is any attacker-controllable input, including:
- query parameters
- path parameters
- request body fields
- headers
- cookies
- uploaded metadata
- values from session-like objects that originated from user input
- stored values previously written by a user or admin
- values returned from other services if they are derived from user-controlled input

### Propagation
Propagation means the input is copied, transformed, normalized, wrapped, or passed through helper functions before reaching the SQL sink.

Do not assume a renamed variable is safe.

### Sink
A sink is the place where SQL is constructed or executed, including:
- raw SQL execution
- query builder raw fragments
- ORM raw helpers
- stored procedure calls with dynamic SQL inside
- helper functions that ultimately build or run SQL
- report/filter systems that translate input into executable SQL

The main audit goal is to determine whether attacker-controlled input can reach a SQL sink in an unsafe way.

## 1.3 Value injection vs structural injection

### Value injection
User input is placed where the query expects a data value.

Example:
- username
- email
- status
- keyword
- numeric ID

This is usually prevented by real parameter binding.

### Structural injection
User input controls SQL structure, not only a value.

Common examples:
- ORDER BY field
- sort direction
- operator
- raw WHERE clause
- table name
- column name
- LIMIT / OFFSET expression
- SELECT field list
- UNION fragment

This is usually **not** solved by normal value placeholders. Structural elements generally require:
- strict allowlists,
- fixed server-side mappings,
- or safe non-string query APIs.

---

# 2. Shared SQL Injection Attack Surfaces

Prioritize these attack surfaces first:

- login and authentication queries
- search and filter endpoints
- reporting and export functionality
- sortable and pageable list endpoints
- admin query tools
- dynamic dashboards
- raw SQL helpers
- saved search / saved filter systems
- analytics queries
- internal tools and legacy endpoints
- background jobs that consume stored user-controlled data

These features frequently involve dynamic conditions, optional filters, or convenience helpers that create SQL injection risk.

---

# 3. Shared Anti-Patterns

These are common danger signals across languages.

## A1. String concatenation into SQL
High-risk pattern:
- `"select ... where x = '" + input + "'"`

Why risky:
The query text is directly influenced by untrusted input.

## A2. Interpolation / format-based SQL
High-risk pattern:
- f-strings
- `%` formatting
- `.format(...)`
- template strings
- string builders

Why risky:
These produce executable query text and do not provide real parameter binding.

## A3. Partial parameterization
High-risk pattern:
- one value is parameterized
- another clause is still concatenated

Example idea:
- `ORDER BY <user-controlled>`
- `LIMIT ?`

Why risky:
A query can still be injectable even if one part is bound safely.

## A4. Raw condition fragments
High-risk pattern:
- user input becomes `WHERE ...`
- user input becomes an operator
- user input becomes arbitrary search expression text

Why risky:
This gives the attacker direct logical control of SQL behavior.

## A5. User-controlled structural elements
High-risk pattern:
- user chooses column
- user chooses table
- user chooses sort field
- user chooses select field list
- user chooses function name or expression

Why risky:
Prepared statements generally do not protect structural SQL elements.

## A6. Misuse of ORM / query builder raw helpers
High-risk pattern:
- raw SQL helper with interpolated input
- unsafe `.raw(...)`, `text(...)`, `whereRaw(...)`, `orderByRaw(...)`, native query APIs

Why risky:
Framework abstractions do not help if raw SQL fragments are still attacker-controlled.

## A7. Second-order SQL injection
High-risk pattern:
- attacker input is stored
- later read back
- later inserted into SQL structure

Why risky:
Input does not need to hit the sink immediately to become dangerous.

## A8. False confidence from escaping
High-risk pattern:
- manual escaping used instead of binding
- developer assumes escaping solves all cases

Why risky:
Escaping is often incomplete, context-sensitive, and easy to misuse.

---

# 4. Shared Protection Model

## 4.1 What strong protection usually looks like

Strong protections typically include:
- prepared statements
- real bound parameters
- ORM filter APIs that do not expose raw SQL text
- safe query builders
- strict allowlists for structural elements
- fixed server-side mappings from user choice to known-safe column names
- separation of "query structure decided by code" from "values provided by user"

## 4.2 What protection does not automatically mean safety

The following do **not** automatically mean the code is safe:
- use of an ORM
- use of a query builder
- use of a framework helper
- use of `prepare(...)` when structural parts are still concatenated
- type casting alone
- escaping alone
- regex validation alone without sink verification

## 4.3 Structural elements need allowlisting
Prepared statements usually protect data values, not structure.

For the following, expect allowlisting or fixed mapping:
- sortable field
- sort direction
- table name
- column name
- operator
- report template selection
- dynamic filter keys

---

# 5. Second-Order SQL Injection

Second-order SQL injection occurs when:
1. attacker input is stored safely or unsafely,
2. the application later reuses that stored value,
3. the stored value influences SQL structure or execution.

Common examples:
- saved filters
- report templates
- stored search conditions
- admin-defined clauses
- user profile fields reused in later query construction
- metadata that later becomes a raw query fragment

Audit questions:
- Can a user control data that is later reused in SQL?
- Is stored data revalidated before it becomes part of a query?
- Is stored data treated as trusted only because it came from the database?

---

# 6. False-Positive Controls

Do not report a vulnerability as `Confirmed` if:
- the input is provably constant or server-controlled,
- the risky string is not actually executed as SQL,
- the structural value is selected from a strict server-side allowlist,
- the sink uses real binding correctly for all untrusted values,
- the apparent sink is unreachable from untrusted input,
- the code pattern is only superficially similar to SQL injection but does not affect executable query text.

Use `Suspected` or `Not enough evidence` if:
- the sink exists but the taint source is unclear,
- the source exists but the final sink is not visible,
- sanitization or binding may exist elsewhere but cannot be confirmed,
- a helper abstracts the final sink and the implementation is not visible,
- stored values may be dangerous but later query usage cannot be verified.

Do not over-claim based only on:
- string concatenation unrelated to SQL,
- presence of raw helper names without tainted input,
- presence of `prepare(...)` alone,
- presence of ORM names alone,
- existence of a dynamic field if it is clearly allowlisted.

---

# 7. Finding Classification

## Confirmed
Use `Confirmed` when there is clear evidence that:
- attacker-controlled input reaches query construction or execution,
- protection is absent, weak, or incomplete,
- and exploitation is plausibly possible.

## Suspected
Use `Suspected` when:
- there is a high-risk pattern,
- the input path or sink path is strongly indicated,
- but a hidden sanitizer, wrapper, or allowlist may exist elsewhere.

## Not enough evidence
Use `Not enough evidence` when:
- critical path visibility is missing,
- the sink or the source cannot be verified,
- or a framework abstraction hides the final behavior.

## Probably safe
Use `Probably safe` when:
- the data flow is visible,
- real parameter binding or strong allowlisting is clearly applied,
- and no attacker-controlled structural influence is evident.

---

# 8. What Good Evidence Looks Like

Strong SQL injection findings usually include:
- the exact entry point
- the tainted input source
- the variable propagation path
- the query construction point
- the execution sink
- the missing parameterization or allowlisting
- the reason attacker input can alter SQL logic or structure

Good findings usually answer:
1. What input is attacker-controlled?
2. Where does it travel?
3. What SQL sink does it reach?
4. What protection should have stopped it?
5. Why can the attacker alter executable behavior?

---

# 9. Shared Remediation Guidance

Preferred fixes include:
- replace raw string assembly with prepared statements
- bind all untrusted values
- move structural choices to fixed server-side mappings
- allowlist sortable columns and directions
- remove raw SQL helpers where safe ORM/query-builder APIs exist
- represent saved filters as structured safe options instead of raw clauses
- revalidate stored values before they influence SQL behavior

Avoid weak fixes such as:
- ad hoc escaping only
- blacklist filtering only
- fragile regex filtering without sink redesign
- assuming numeric-looking input is always safe

---

# 10. Shared Quick Checklist

Use this as a reminder, not as a substitute for reasoning.

- Is there a real attacker-controlled source?
- Does it reach a SQL construction or execution sink?
- Is the query built with concatenation, interpolation, or raw fragments?
- Are all user-controlled values bound with real parameters?
- Can user input control SQL structure?
- Is there a strict allowlist for structural elements?
- Can stored user data later become executable SQL?
- Are export/report/admin paths as safe as normal paths?
