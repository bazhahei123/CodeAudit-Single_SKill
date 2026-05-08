# PHP SQL Injection Cases

## Purpose

This file contains PHP-specific SQL injection patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in PHP, especially in:
- Laravel
- Symfony
- ThinkPHP
- Yii
- CodeIgniter
- raw PDO / mysqli code
- PHP backends exposing REST APIs, admin panels, reporting systems, or search functionality

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real query sink in the target code.

---

# 1. PHP SQL Control Points

When auditing PHP applications, prioritize these control points.

## 1.1 Route and entry-point controls
Look for:
- Laravel routes
- controller methods
- legacy PHP entry files
- AJAX endpoints
- admin panels
- API endpoints
- background/admin job triggers

## 1.2 Query construction points
Look for:
- string concatenation
- interpolation into SQL strings
- query builder `orderBy`, raw fragments
- `DB::raw(...)`
- raw PDO / mysqli queries
- helper methods building WHERE / ORDER fragments

## 1.3 Query execution sinks
Look for:
- `$pdo->query(...)`
- `$pdo->prepare(...)`
- `$stmt->execute(...)`
- `mysqli_query(...)`
- `DB::select(...)`
- `DB::statement(...)`
- Eloquent raw helpers
- repository helpers using raw SQL

## 1.4 ORM / framework safety controls
Look for:
- prepared statements
- bound parameters
- Eloquent safe query APIs
- Laravel query builder
- Symfony DBAL parameter binding
- allowlists for sortable fields and operators

## 1.5 Stored and second-order data flows
Look for:
- saved filters
- report templates
- admin-configured raw conditions
- stored values later reused in raw SQL

---

# 2. High-Coverage PHP SQL Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, query construction behavior, sink execution, and missing binding or structural controls.

## 2.1 Web, framework, and request entry candidates
Search for:
- `Route::get`
- `Route::post`
- `Route::put`
- `Route::patch`
- `Route::delete`
- `Route::any`
- `Route::match`
- `Route::resource`
- `Route::apiResource`
- `Controller`
- `__invoke`
- `Request $request`
- `$request->input`
- `$request->get`
- `$request->query`
- `$request->post`
- `$request->all`
- `$request->cookie`
- `$request->header`
- `$request->getContent`
- `$_GET`
- `$_POST`
- `$_REQUEST`
- `$_COOKIE`
- `php://input`
- `#[Route]`
- `@Route`
- `AbstractController`
- `Action`
- `Middleware`

## 2.2 Queue, command, admin, report, and export entries
Search for:
- `ShouldQueue`
- `handle`
- `Job`
- `Listener`
- `EventSubscriber`
- `Command`
- `Console`
- `schedule`
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
- `generateExport`
- `DataTables`
- `ajax`

## 2.3 Query construction and structural fragment candidates
Search for:
- `$sql`
- `$query`
- `$statement`
- `$where`
- `$whereClause`
- `$condition`
- `$filter`
- `$orderBy`
- `$sort`
- `$sortField`
- `$direction`
- `$column`
- `$table`
- `$operator`
- `$limit`
- `$offset`
- string concatenation with `.`
- `sprintf`
- `implode`
- interpolated SQL strings
- `DB::raw`
- `whereRaw`
- `orWhereRaw`
- `orderByRaw`
- `groupByRaw`
- `havingRaw`
- `selectRaw`
- `fromRaw`
- `custom_query`
- `report_sql`

## 2.4 PDO, mysqli, and raw database sink candidates
Search for:
- `$pdo->query`
- `$pdo->prepare`
- `$stmt->execute`
- `PDO::query`
- `PDO::prepare`
- `mysqli_query`
- `mysqli_multi_query`
- `$mysqli->query`
- `$mysqli->multi_query`
- `$conn->executeQuery`
- `$conn->executeStatement`
- `executeQuery`
- `executeStatement`
- `query`
- `prepare`
- `fetchAll`
- `pg_query`
- `pg_send_query`
- `sqlite_query`
- `oci_parse`
- `sqlsrv_query`

## 2.5 Framework ORM and query-builder raw sink candidates
Search for:
- `DB::select`
- `DB::statement`
- `DB::unprepared`
- `DB::insert`
- `DB::update`
- `DB::delete`
- `DB::table`
- `DB::raw`
- `whereRaw`
- `orWhereRaw`
- `orderByRaw`
- `groupByRaw`
- `havingRaw`
- `selectRaw`
- `fromRaw`
- `joinSub`
- `Eloquent`
- `Model::query`
- `createQueryBuilder`
- `Doctrine\\DBAL`
- `createNativeQuery`
- `getConnection`
- `ThinkPHP`
- `Yii::$app->db->createCommand`
- `CodeIgniter`

## 2.6 Required-control candidates
Search near sinks for:
- `prepare`
- `bindParam`
- `bindValue`
- `execute([`
- named placeholders
- positional placeholders
- query builder safe methods
- `where`
- `orderBy` with mapping
- `Rule::in`
- `Validator`
- `validate`
- `enum`
- `match`
- `allowedSorts`
- `allowedColumns`
- `allowedTables`
- `Arr::only`
- `in_array` strict
- `filter_var`
- `Doctrine QueryBuilder parameters`
- `setParameter`
- `ExpressionBuilder`

## 2.7 PHP graph search recipes
Useful combinations:

```text
Route::get + DB::select
$request->input + DB::raw
$_GET + mysqli_query
Controller + whereRaw
orderByRaw + request parameter
PDO::query + string concatenation
DB::statement + interpolated variable
savedFilter + whereClause + DB::select
reportTemplate + custom_query + query
createCommand + request parameter
```

---

# 3. PHP SQL Injection Anti-Patterns

## 3.1 Raw SQL Anti-Patterns

### A1. Concatenated SQL in PDO / mysqli
```php
$sql = "select * from users where username = '" . $username . "'";
$result = $pdo->query($sql);
```

### A2. Interpolated SQL string
```php
$sql = "select * from orders where status = '$status'";
$result = mysqli_query($conn, $sql);
```

## 3.2 Structural Injection Anti-Patterns

### A3. User-controlled ORDER BY
```php
$sql = "select * from products order by " . $sort;
$rows = $pdo->query($sql)->fetchAll();
```

### A4. Raw WHERE fragment
```php
$sql = "select * from invoice where " . $condition;
$rows = $pdo->query($sql)->fetchAll();
```

## 3.3 Laravel / Query Builder Misuse Anti-Patterns

### A5. `DB::raw(...)` with request-derived fragment
```php
$query = DB::table('users')->orderByRaw($sort);
```

### A6. Raw select with interpolated input
```php
$rows = DB::select("select * from report where type = '$type'");
```

## 3.4 Mixed Safe / Unsafe Anti-Patterns

### A7. Partial parameterization
```php
$sql = "select * from logs order by " . $sort . " limit ?";
$stmt = $pdo->prepare($sql);
$stmt->execute([$limit]);
```

### A8. Dynamic table or column names
```php
$sql = "select " . $column . " from " . $table . " where id = ?";
$stmt = $pdo->prepare($sql);
$stmt->execute([$id]);
```

## 3.5 Second-Order Anti-Patterns

### A9. Stored filter reused in SQL
```php
$where = $savedFilter->where_clause;
$sql = "select * from logs where " . $where;
$rows = DB::select($sql);
```

### A10. Legacy admin helper using saved raw query
```php
$sql = $report->custom_query;
return $pdo->query($sql)->fetchAll();
```

---

# 4. Case Templates

## Case H-SQL-1: Raw Query Injection

### Vulnerable pattern
```php
$sql = "select * from users where username = '" . $username . "'";
$result = $pdo->query($sql);
```

### Safer pattern
```php
$stmt = $pdo->prepare("select * from users where username = ?");
$stmt->execute([$username]);
```

## Case H-SQL-2: Laravel Raw Fragment Injection

### Vulnerable pattern
```php
$query = DB::table('users')->orderByRaw($sort);
```

## Case H-SQL-3: Structural Injection

### Vulnerable pattern
```php
$sql = "select * from products order by " . $sort;
$rows = $pdo->query($sql)->fetchAll();
```

## Case H-SQL-4: Second-Order Injection

### Vulnerable pattern
```php
$where = $savedFilter->where_clause;
$sql = "select * from logs where " . $where;
$rows = DB::select($sql);
```

---

# 5. PHP-Specific Audit Heuristics

## 4.1 Laravel heuristics
Pay attention to:
- `DB::select(...)`
- `DB::statement(...)`
- `DB::raw(...)`
- `orderByRaw(...)`
- `whereRaw(...)`
- raw Eloquent scopes or repository helpers
- interpolated SQL inside controllers or services

## 4.2 PDO / mysqli heuristics
Pay attention to:
- direct `query(...)`
- string interpolation before `prepare(...)`
- partial placeholder usage
- manual escaping instead of binding
- helper methods constructing SQL strings centrally

## 4.3 Symfony / DBAL heuristics
Pay attention to:
- raw SQL through DBAL connection objects
- `executeQuery(...)`
- concatenated SQL fragments
- dynamic order or filter pieces
- repository helpers returning raw SQL

## 4.4 Structural control heuristics
Check whether the following are user-controlled:
- sortable field
- sort direction
- table name
- column name
- operator
- filter clause
- limit / offset expression
- union-capable fragments

## 4.5 Layer inconsistency heuristics
Check whether SQL safety is consistent across:
- search vs export/report endpoints
- web routes vs admin/internal routes
- controller vs service vs repository code
- modern code vs legacy helper methods
- GET vs POST routes reaching the same sink

---

# 6. False-Positive Controls

Do not report a vulnerability as confirmed if:
- the input is provably constant or server-controlled,
- the risky string is not actually executed as SQL,
- structural fields are restricted through a strong allowlist,
- safe query APIs are used correctly for all untrusted values,
- the apparent sink is unreachable from untrusted input.

---

# 7. What Good Evidence Looks Like

Strong evidence usually includes:
- the exact entry point
- the tainted input source
- the query construction point
- the query execution sink
- the missing or weak parameterization / allowlisting
- the reason attacker input can alter SQL logic or structure

---

# 8. Quick PHP SQL Audit Checklist

- Do request inputs reach PDO, mysqli, DB::select, DB::raw, or raw query helpers?
- Are queries built with concatenation, interpolation, or raw fragments?
- Are all untrusted values bound through prepared execution safely?
- Are ORDER BY / LIMIT / column / table controls allowlisted?
- Are raw Laravel / DBAL helpers used safely?
- Can stored user data later become executable SQL?
- Are export/report/admin queries as safe as primary user queries?
