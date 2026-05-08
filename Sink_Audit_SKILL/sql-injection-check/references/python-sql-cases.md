# Python SQL Injection Cases

## Purpose

This file contains Python-specific SQL injection patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Python, especially in:
- Django
- Django REST Framework
- Flask
- FastAPI
- Starlette
- SQLAlchemy-based services
- raw DB-API code
- Python backends exposing REST APIs, admin panels, reporting systems, or search functionality

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real query sink in the target code.

---

# 1. Python SQL Control Points

When auditing Python applications, prioritize these control points.

## 1.1 Route and entry-point controls
Look for:
- Django views
- DRF APIViews and ViewSets
- Flask route handlers
- FastAPI routers
- GraphQL resolvers
- background jobs fed by request data

## 1.2 Query construction points
Look for:
- f-strings
- `%` formatting
- `.format(...)`
- string concatenation
- raw SQL helpers
- ORM `.raw(...)`
- SQLAlchemy `text(...)`
- helper methods building WHERE / ORDER fragments

## 1.3 Query execution sinks
Look for:
- `cursor.execute(...)`
- `cursor.executemany(...)`
- Django raw queries
- SQLAlchemy `.execute(...)`
- `.raw(...)`
- helper methods around database cursors

## 1.4 ORM / framework safety controls
Look for:
- Django ORM filters
- SQLAlchemy parameter binding
- DB-API placeholders
- allowlists for sortable fields and operators
- safe translation layers from user options to fixed query fragments

## 1.5 Stored and second-order data flows
Look for:
- saved search rules
- report templates
- stored filters
- admin-configured conditions later reused in SQL

---

# 2. High-Coverage Python SQL Candidate Inventory

Use these candidates as search seeds for graph-database or taint-tracking workflows. A match is not a finding by itself; confirm attacker influence, query construction behavior, sink execution, and missing binding or structural controls.

## 2.1 Web, API, and request entry candidates
Search for:
- `@app.route`
- `@blueprint.route`
- `@bp.route`
- `Flask`
- `request.args`
- `request.form`
- `request.values`
- `request.get_json`
- `request.cookies`
- `request.headers`
- `@api_view`
- `APIView`
- `ViewSet`
- `ModelViewSet`
- `GenericAPIView`
- `path(`
- `re_path(`
- `View`
- `dispatch`
- `get`
- `post`
- `put`
- `patch`
- `delete`
- `@app.get`
- `@app.post`
- `FastAPI`
- `APIRouter`
- `Query`
- `Path`
- `Body`
- `Request`
- `GraphQLView`
- `Mutation`

## 2.2 Worker, message, admin, report, and export entries
Search for:
- `@shared_task`
- `@app.task`
- `Celery`
- `dramatiq.actor`
- `rq.job`
- `BaseCommand`
- `handle`
- `KafkaConsumer`
- `basic_consume`
- `on_message`
- `admin`
- `dashboard`
- `analytics`
- `report`
- `export`
- `search`
- `filter`
- `sort`
- `saved_filter`
- `report_template`
- `run_report`
- `generate_export`

## 2.3 Query construction and structural fragment candidates
Search for:
- `sql`
- `query`
- `statement`
- `where_clause`
- `condition`
- `filter_clause`
- `order_by`
- `sort`
- `sort_field`
- `sort_direction`
- `column`
- `table`
- `operator`
- `limit`
- `offset`
- `f"select`
- `f" SELECT`
- `.format(`
- `%`
- `" +`
- `" ".join`
- `text(f`
- `raw(`
- `literal_column`
- `column(`
- `table(`
- `desc(`
- `asc(`
- `saved_filter`
- `custom_query`
- `report_sql`

## 2.4 DB-API and raw cursor sink candidates
Search for:
- `cursor.execute`
- `cursor.executemany`
- `cursor.executescript`
- `connection.execute`
- `conn.execute`
- `db.execute`
- `engine.execute`
- `execute_query`
- `run_query`
- `raw_query`
- `psycopg2`
- `psycopg`
- `pymysql`
- `MySQLdb`
- `mysql.connector`
- `sqlite3`
- `aiosqlite`
- `asyncpg`
- `pyodbc`
- `cx_Oracle`
- `oracledb`
- `mssql`

## 2.5 Django, SQLAlchemy, and ORM raw sink candidates
Search for:
- `objects.raw`
- `.raw(`
- `RawSQL`
- `extra(`
- `.extra`
- `connection.cursor`
- `django.db.connection`
- `QuerySet.order_by`
- `annotate`
- `Func`
- `SQLAlchemy`
- `text(`
- `.execute(`
- `session.execute`
- `select`
- `literal_column`
- `column`
- `table`
- `from_statement`
- `order_by(text`
- `filter(text`
- `where(text`
- `bindparam`
- `sqlalchemy.sql`
- `tortoise.raw`
- `peewee.SQL`

## 2.6 Required-control candidates
Search near sinks for:
- `%s` placeholder with parameter tuple/list
- `?` placeholder with parameter tuple/list
- `:name` bind parameter
- `bindparam`
- `params=`
- `execute(sql, params)`
- `RawSQL(..., params)`
- `text(...).bindparams`
- ORM `.filter`
- `Q(`
- `safe_order_fields`
- `allowed_sort_fields`
- `allowed_columns`
- `allowed_tables`
- `Enum`
- `Literal`
- `choices`
- `re.fullmatch`
- `validate_sort`
- `schema`
- `pydantic`

## 2.7 Python graph search recipes
Useful combinations:

```text
@app.route + cursor.execute
request.args + f"select
FastAPI + text(f
APIView + objects.raw
ViewSet + connection.cursor
@shared_task + raw SQL
order_by + request parameter
literal_column + request parameter
saved_filter + where_clause + execute
report_template + session.execute
cursor.executescript + request
```

---

# 3. Python SQL Injection Anti-Patterns

## 3.1 Raw DB-API Anti-Patterns

### A1. Concatenated SQL in `execute`
```python
sql = "select * from users where username = '" + username + "'"
cursor.execute(sql)
```

### A2. f-string SQL
```python
sql = f"select * from orders where status = '{status}'"
cursor.execute(sql)
```

## 3.2 Structural Injection Anti-Patterns

### A3. User-controlled ORDER BY
```python
sql = f"select * from products order by {sort}"
cursor.execute(sql)
```

### A4. Raw WHERE fragment
```python
sql = "select * from invoice where " + condition
cursor.execute(sql)
```

## 3.3 Django / ORM Misuse Anti-Patterns

### A5. Django `.raw(...)` with interpolation
```python
query = f"select * from app_user where name = '{name}'"
User.objects.raw(query)
```

### A6. Raw cursor execution inside Django view
```python
cursor.execute("select * from orders where id = " + order_id)
```

## 3.4 SQLAlchemy / Text Anti-Patterns

### A7. Unsafe `text(...)` usage
```python
stmt = text(f"select * from report where type = '{report_type}'")
db.execute(stmt)
```

### A8. Mixed safe and unsafe query fragments
```python
stmt = text("select * from logs order by " + sort + " limit :limit")
db.execute(stmt, {"limit": limit})
```

## 3.5 Second-Order Anti-Patterns

### A9. Stored filter reused in SQL
```python
where_clause = saved_filter.where_clause
sql = "select * from logs where " + where_clause
cursor.execute(sql)
```

### A10. Dynamic table or column names
```python
sql = f"select {column} from {table} where id = %s"
cursor.execute(sql, [record_id])
```

---

# 4. Case Templates

## Case P-SQL-1: Raw Execute Injection

### Vulnerable pattern
```python
sql = "select * from users where username = '" + username + "'"
cursor.execute(sql)
```

### Safer pattern
```python
cursor.execute("select * from users where username = %s", [username])
```

## Case P-SQL-2: Django Raw Query Injection

### Vulnerable pattern
```python
query = f"select * from app_user where name = '{name}'"
User.objects.raw(query)
```

## Case P-SQL-3: Structural Injection

### Vulnerable pattern
```python
sql = f"select * from products order by {sort}"
cursor.execute(sql)
```

## Case P-SQL-4: Second-Order Injection

### Vulnerable pattern
```python
where_clause = saved_filter.where_clause
sql = "select * from logs where " + where_clause
cursor.execute(sql)
```

---

# 5. Python-Specific Audit Heuristics

## 4.1 Django / DRF heuristics
Pay attention to:
- `.raw(...)`
- `connection.cursor()`
- raw SQL helpers in views or services
- queryset fallbacks to custom SQL
- report/export endpoints using manual SQL

## 4.2 Flask / FastAPI heuristics
Pay attention to:
- f-strings in route handlers
- `.format(...)` in query helpers
- service-layer SQL builder functions
- dependency-injected values later used in SQL strings

## 4.3 SQLAlchemy heuristics
Pay attention to:
- `text(...)`
- `.execute(...)`
- dynamic `order_by` using raw strings
- unsafe `literal_column(...)` or manual text fragments
- mixed safe/unsafe statements

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
- service vs repository/helper code
- modern code vs legacy SQL helpers
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

# 8. Quick Python SQL Audit Checklist

- Do request inputs reach cursor execution, raw SQL, or ORM raw helpers?
- Are queries built with concatenation, f-strings, format strings, or raw fragments?
- Are all untrusted values bound through the driver or ORM safely?
- Are ORDER BY / LIMIT / column / table controls allowlisted?
- Are Django `.raw(...)` or SQLAlchemy `text(...)` used safely?
- Can stored user data later become executable SQL?
- Are export/report/admin queries as safe as primary user queries?
