# Java Unsafe Deserialization Cases

## Purpose

This file contains Java-specific unsafe deserialization patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- Java native serialization
- XMLDecoder
- Jackson object mapping
- Fastjson
- Hessian / Kryo / custom binary protocol code
- Java backends exposing import, RPC, cache, session, or object-restore functionality

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real data flow, trust boundary, and dangerous post-deserialization behavior in the target code.

---

# 1. Java Deserialization Control Points

When auditing Java applications, prioritize these control points.

## 1.1 Entry and restore points
Look for:
- `ObjectInputStream`
- `readObject`
- `readUnshared`
- `XMLDecoder`
- custom binary decoders
- framework message converters
- RPC decoders
- session or cache restore helpers
- import handlers

## 1.2 Object behavior controls
Look for:
- `readObject`
- `readResolve`
- `readExternal`
- `Externalizable`
- lifecycle callbacks
- classes with side effects during restore or cleanup
- downstream code that trusts restored object state

## 1.3 Framework and library controls
Look for:
- Jackson default typing or polymorphic type handling
- Fastjson auto-type style behavior
- serializer wrappers
- cache or session object storage
- internal protocol handlers
- allowlist / denylist configuration

## 1.4 Trust-boundary controls
Look for:
- signed or unsigned serialized cookies
- import payload validation
- queue or cache trust assumptions
- internal-only object transport assumptions
- stored blob replay or reprocessing

---

# 2. Java Unsafe Deserialization Anti-Patterns

### A1. Native deserialization on request-derived data
```java
ObjectInputStream ois = new ObjectInputStream(inputStream);
Object obj = ois.readObject();
```

Why risky:
Untrusted input reaches native object restoration directly.

### A2. XMLDecoder on untrusted input
```java
XMLDecoder decoder = new XMLDecoder(inputStream);
Object obj = decoder.readObject();
```

Why risky:
`XMLDecoder` is dangerous when fed attacker-controlled content.

### A3. Polymorphic object mapping without strict controls
```java
Object obj = mapper.readValue(json, Object.class);
```

Why risky:
Unsafe type resolution may restore dangerous or unintended types depending on configuration.

### A4. Stored blob later deserialized
```java
byte[] blob = cache.get(key);
ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(blob));
Object obj = ois.readObject();
```

Why risky:
The attacker may influence the stored blob indirectly, creating a second-order path.

### A5. Implicit trust in internal queue or RPC payload
```java
Message msg = queueConsumer.receive();
Object obj = serializer.deserialize(msg.getBody());
```

Why risky:
Internal transport does not automatically imply trusted origin.

---

# 3. Case Templates

## Case J-DESER-1: Direct native deserialization

### Vulnerable pattern
```java
ObjectInputStream ois = new ObjectInputStream(inputStream);
Object obj = ois.readObject();
```

### Audit focus
Verify whether `inputStream` is attacker-controlled or weakly trusted, and whether dangerous classes or gadget behavior are reachable.

## Case J-DESER-2: XMLDecoder usage

### Vulnerable pattern
```java
XMLDecoder decoder = new XMLDecoder(inputStream);
Object obj = decoder.readObject();
```

### Audit focus
Treat attacker influence on `inputStream` as high risk.

## Case J-DESER-3: Polymorphic object mapping

### Vulnerable pattern
```java
Object obj = mapper.readValue(payload, Object.class);
```

### Audit focus
Verify whether type restoration is constrained by a strict allowlist or safe DTO model.

## Case J-DESER-4: Second-order cache / stored blob restore

### Vulnerable pattern
```java
byte[] blob = cache.get(key);
Object obj = serializer.deserialize(blob);
```

### Audit focus
Verify who controls the blob and whether integrity or type restrictions exist.

---

# 4. Java-Specific Audit Heuristics

## 4.1 Native serialization heuristics
Pay attention to:
- `ObjectInputStream`
- `readObject`
- `readUnshared`
- `Externalizable`
- restore helpers wrapping native serialization

## 4.2 Dangerous lifecycle heuristics
Pay attention to:
- custom `readObject`
- `readResolve`
- `readExternal`
- side-effect-heavy classes
- restore-time callbacks or initialization logic

## 4.3 Jackson / mapper heuristics
Pay attention to:
- default typing
- polymorphic type metadata
- `Object.class` or permissive base-type restore
- unsafe mapper configuration
- helper wrappers that hide mapper behavior

## 4.4 Fastjson / alternative serializer heuristics
Pay attention to:
- auto-type behavior
- parser features enabling broad type resolution
- wrapper libraries exposing unsafe defaults

## 4.5 Layer inconsistency heuristics
Check whether deserialization safety is consistent across:
- request path vs queue consumer
- import path vs cache/session restore
- public API vs internal tooling
- direct restore helper vs framework wrapper
