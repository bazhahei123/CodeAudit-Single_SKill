# Java SQL Injection Cases

## Purpose

This file contains Java-specific SQL injection patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- Spring JDBC
- MyBatis
- JPA / Hibernate
- jOOQ
- Servlet-based web applications
- Java backends exposing REST APIs, admin panels, reporting systems, or search functionality

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow and real query sink in the target code.

---

# 1. Java SQL Control Points

When auditing Java applications, prioritize these control points.

## 1.1 Route and entry-point controls
Look for:
- `@RestController`
- `@Controller`
- `@RequestMapping`
- `@GetMapping`
- `@PostMapping`
- `@RequestParam`
- `@PathVariable`
- request body DTOs
- GraphQL resolvers
- RPC-style handlers

Questions:
- Which routes accept search, filter, sort, pagination, reporting, or login input?
- Which entry points accept identifiers, keywords, column names, sort fields, or raw conditions?
- Are admin or internal reporting routes clearly separated from public/user routes?

## 1.2 Query construction points
Look for:
- string concatenation in services or DAOs
- `StringBuilder` / `String.format`
- HQL / JPQL string assembly
- MyBatis dynamic SQL
- `@Query`
- raw JDBC query strings
- helper methods building WHERE/ORDER fragments

Questions:
- Is user input inserted directly into SQL, JPQL, HQL, or query fragments?
- Is only part of the query parameterized while other structural parts remain dynamic?
- Are dynamic filters or order clauses built from request parameters?

## 1.3 Query execution sinks
Look for:
- `Statement.executeQuery`
- `Statement.execute`
- `PreparedStatement`
- `JdbcTemplate.query(...)`
- `JdbcTemplate.update(...)`
- `EntityManager.createQuery(...)`
- `EntityManager.createNativeQuery(...)`
- MyBatis mapper methods
- repository helpers using raw SQL

Questions:
- Which APIs execute the final query?
- Does the sink support bound parameters?
- Is the safe API actually used safely?

## 1.4 ORM / framework safety controls
Look for:
- prepared statements
- named parameters
- positional parameters
- MyBatis `#{}` vs `${}`
- JPA parameter binding
- jOOQ typed query building
- allowlists for sortable columns and operators

Questions:
- Are placeholders used for all user-controlled values?
- Are structural SQL fragments restricted by allowlists?
- Is `${}` used where `#{}` should be used?

## 1.5 Stored and second-order data flows
Look for:
- user input saved to database and later reused in SQL fragments
- saved search rules
- report templates
- dynamic admin filters
- stored sort fields, conditions, or table names

Questions:
- Can previously stored data later influence SQL structure?
- Are stored values trusted without revalidation?

---

# 2. Java SQL Injection Anti-Patterns

## 2.1 Raw JDBC Anti-Patterns

### A1. Concatenated SQL in `Statement`
```java
String sql = "select * from users where username = '" + username + "'";
ResultSet rs = statement.executeQuery(sql);
```

Why risky:
User input is inserted directly into SQL and executed without binding.

### A2. Partial parameterization with unsafe structure
```java
String sql = "select * from orders order by " + sort + " limit ?";
PreparedStatement ps = conn.prepareStatement(sql);
ps.setInt(1, limit);
```

Why risky:
Even if one value is parameterized, the ORDER BY structure remains user-controlled.

## 2.2 Spring JDBC Anti-Patterns

### A3. `JdbcTemplate` with concatenated query
```java
String sql = "select * from product where name like '%" + keyword + "%'";
return jdbcTemplate.queryForList(sql);
```

Why risky:
The query is executed through a framework helper, but the SQL string is still unsafe.

### A4. Dynamic raw WHERE fragment
```java
String sql = "select * from invoice where " + condition;
return jdbcTemplate.query(sql, mapper);
```

Why risky:
A raw condition fragment can give the attacker direct control over SQL logic.

## 2.3 MyBatis Anti-Patterns

### A5. `${}` used with request-derived data
```xml
<select id="listUsers" resultType="User">
  select * from users order by ${sort}
</select>
```

Why risky:
`${}` performs textual substitution, not safe binding.

### A6. Raw WHERE fragment in XML
```xml
<select id="search" resultType="Order">
  select * from orders where ${whereClause}
</select>
```

Why risky:
The query structure is directly controlled by the caller.

## 2.4 JPA / Hibernate Anti-Patterns

### A7. JPQL built with string concatenation
```java
String q = "from User u where u.name = '" + name + "'";
return entityManager.createQuery(q).getResultList();
```

Why risky:
JPQL/HQL injection can alter query logic just like raw SQL injection.

### A8. Native query with unsafe interpolation
```java
String sql = "select * from report where type = '" + type + "'";
return entityManager.createNativeQuery(sql).getResultList();
```

Why risky:
Native query execution with interpolated input is a direct injection sink.

## 2.5 Second-Order and Structural Anti-Patterns

### A9. Stored rule reused in SQL
```java
String where = savedFilter.getWhereClause();
String sql = "select * from logs where " + where;
return jdbcTemplate.query(sql, mapper);
```

Why risky:
Even if the original input was stored earlier, it may still control later SQL execution.

### A10. Dynamic table or column names
```java
String sql = "select " + column + " from " + table + " where id = ?";
PreparedStatement ps = conn.prepareStatement(sql);
ps.setLong(1, id);
```

Why risky:
Prepared statements do not protect structural elements like table or column names.

---

# 3. Case Templates

## Case J-SQL-1: Raw JDBC Injection

### Vulnerable pattern
```java
String sql = "select * from users where username = '" + username + "'";
ResultSet rs = statement.executeQuery(sql);
```

### Safer pattern
```java
String sql = "select * from users where username = ?";
PreparedStatement ps = conn.prepareStatement(sql);
ps.setString(1, username);
```

## Case J-SQL-2: MyBatis Structural Injection

### Vulnerable pattern
```xml
<select id="listUsers" resultType="User">
  select * from users order by ${sort}
</select>
```

## Case J-SQL-3: JPA / Hibernate Injection

### Vulnerable pattern
```java
String q = "from User u where u.name = '" + name + "'";
return entityManager.createQuery(q).getResultList();
```

### Safer pattern
```java
return entityManager
    .createQuery("from User u where u.name = :name")
    .setParameter("name", name)
    .getResultList();
```

## Case J-SQL-4: Second-Order Injection

### Vulnerable pattern
```java
String where = savedFilter.getWhereClause();
String sql = "select * from logs where " + where;
return jdbcTemplate.query(sql, mapper);
```

---

# 4. Java-Specific Audit Heuristics

## 4.1 Spring JDBC heuristics
Pay attention to:
- `JdbcTemplate.query(...)`
- `JdbcTemplate.queryForList(...)`
- `JdbcTemplate.update(...)`
- whether SQL strings are fully static
- whether placeholders are used correctly
- whether sort/filter fragments are built separately and unsafely

## 4.2 MyBatis heuristics
Pay attention to:
- `${}` usage
- dynamic XML fragments
- provider-based SQL builders
- `ORDER BY`, `WHERE`, `LIMIT` fragments
- mapper methods taking raw strings

## 4.3 JPA / Hibernate heuristics
Pay attention to:
- `createQuery(...)`
- `createNativeQuery(...)`
- string-built JPQL/HQL
- concatenated native queries
- parameter binding with named or positional parameters

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
- service vs DAO/repository code
- modern code vs legacy helper methods
- GET vs POST routes reaching the same sink

---

# 5. False-Positive Controls

Do not report a vulnerability as confirmed if:
- the input is provably constant or server-controlled,
- the risky string is not actually executed as SQL,
- structural fields are restricted through a strong allowlist,
- safe query APIs are used correctly for all untrusted values,
- the apparent sink is unreachable from untrusted input.

---

# 6. What Good Evidence Looks Like

Strong evidence usually includes:
- the exact entry point
- the tainted input source
- the query construction point
- the query execution sink
- the missing or weak parameterization / allowlisting
- the reason attacker input can alter SQL logic or structure

---

# 7. Quick Java SQL Audit Checklist

- Do request inputs reach JDBC, MyBatis, JPA, or raw SQL helpers?
- Are queries built with concatenation, format strings, or raw fragments?
- Are all untrusted values bound with prepared statements?
- Are ORDER BY / LIMIT / column / table controls allowlisted?
- Is `${}` used with request-derived data in MyBatis?
- Are native queries and JPQL/HQL built safely?
- Can stored user data later become executable SQL?
- Are export/report/admin queries as safe as primary user queries?
