# Java Unsafe Deserialization Source Cases

## Purpose

This file contains Java-specific source point patterns and audit cases for unsafe deserialization source discovery.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- Java native serialization
- XMLDecoder
- Jackson object mapping
- Fastjson
- Hessian / Kryo / custom binary protocol code
- Java backends exposing import, RPC, cache, session, or object-restore functionality

This reference is guidance, not proof. Do not report a vulnerability only because a source resembles one of the cases below. Always verify the real source origin, propagation path, trust boundary, and downstream restore behavior in the target code.

---

# 1. Java Source Discovery Points

Prioritize these source values and events:
- `InputStream`, `byte[]`, `String`, `ByteBuffer`, and request body payloads
- `@RequestBody`, `@RequestParam`, `@RequestHeader`, cookies, multipart files, and uploaded metadata
- cache values, session attributes, Redis values, and stored blobs
- queue message bodies, RPC frames, and internal protocol payloads
- Jackson/Fastjson type metadata such as `@type`, class name, or polymorphic discriminator fields
- XML content passed toward `XMLDecoder`
- binary payloads passed toward Hessian, Kryo, Java serialization, or custom decoders
- object fields restored into lifecycle methods or post-restore business logic

Source questions:
- Which source supplies bytes or text to object restoration?
- Is the source client-controlled, external-system-controlled, stored attacker-influenced, or server-trusted?
- Are class names or type metadata influenced by the source?
- Is the source decoded, decompressed, or loaded from storage before restore?
- Which follow-up sink should inspect `ObjectInputStream`, `XMLDecoder`, object mapper, or serializer wrappers?

---

# 2. Java Source Patterns

## J-S1. Request-derived stream or bytes
Example idea:
- request body, upload stream, or multipart file becomes an `InputStream` or `byte[]`.

Audit relevance:
Native Java serialization, XMLDecoder, and binary decoders often consume streams or byte arrays.

Follow-up:
- verify whether the stream reaches `ObjectInputStream`, `XMLDecoder`, Kryo, Hessian, or custom object decoders.

## J-S2. Encoded serialized payload
Example idea:
- request value is base64-decoded, decompressed, or converted to bytes before restore.

Audit relevance:
Transformation does not change attacker influence unless integrity and type constraints are proven.

Follow-up:
- trace the decoded value into restore helpers or mappers.

## J-S3. Polymorphic type metadata
Example idea:
- JSON contains `@type`, class name, discriminator, or mapper metadata.

Audit relevance:
Type metadata may influence what class is restored or instantiated.

Follow-up:
- inspect mapper configuration, default typing, auto-type behavior, and allowlists.

## J-S4. Stored cache/session blob
Example idea:
- Redis/cache/session/database value is read as bytes or string and restored later.

Audit relevance:
Stored data can be second-order source material if attacker influence exists upstream.

Follow-up:
- identify who can write the blob and whether integrity or trusted-origin controls exist.

## J-S5. Queue or RPC payload
Example idea:
- message body, RPC frame, or internal event is passed to serializer helpers.

Audit relevance:
Internal transport is an external-system-controlled or mixed source until producer trust and integrity are proven.

Follow-up:
- verify producer control, event authenticity, and safe deserializer settings.

## J-S6. Restore-relevant object state
Example idea:
- restored object fields include file paths, URLs, commands, templates, or callback-like values.

Audit relevance:
These fields may later feed lifecycle hooks or trusted post-restore logic.

Follow-up:
- inspect `readObject`, `readResolve`, `readExternal`, lifecycle callbacks, and post-restore consumers.

---

# 3. Case Templates

## Case J-S-DESER-1: Native serialization source

Source focus:
Identify the origin of the `InputStream`, byte array, or request/file payload that may be passed into `ObjectInputStream`.

Recommended follow-up:
Verify trusted origin, object/type constraints, and reachable lifecycle behavior.

## Case J-S-DESER-2: XMLDecoder source

Source focus:
Identify XML content, stream origin, upload source, or stored XML blob passed toward `XMLDecoder`.

Recommended follow-up:
Verify whether the XML source is attacker-controlled and whether XMLDecoder can be avoided or constrained.

## Case J-S-DESER-3: Polymorphic mapper source

Source focus:
Identify type metadata, `Object.class` mapper targets, default typing, and request/stored JSON origin.

Recommended follow-up:
Verify strict DTO mapping, safe mapper config, and type allowlists.

## Case J-S-DESER-4: Second-order cache or queue source

Source focus:
Identify cache key, stored blob, queue producer, message body, and later restore helper.

Recommended follow-up:
Verify attacker influence, integrity controls, and safe deserializer behavior.

---

# 4. Java-Specific Audit Heuristics

## 4.1 Native serialization source heuristics
Pay attention to:
- `InputStream` origin
- `ByteArrayInputStream` data source
- `ObjectInputStream` wrappers
- `readObject` helper methods
- uploaded or request-derived bytes

## 4.2 Jackson / Fastjson source heuristics
Pay attention to:
- `@type`
- class name fields
- polymorphic discriminator fields
- `Object.class` targets
- default typing and auto-type source values
- mapper wrappers that hide source-to-type behavior

## 4.3 XML and binary decoder source heuristics
Pay attention to:
- XML text origin
- uploaded XML or stored XML blobs
- Hessian/Kryo/custom protocol byte sources
- RPC and internal protocol frame origins

## 4.4 Cache / session / queue source heuristics
Pay attention to:
- Redis or cache value writers
- session serialization formats
- queue message producers
- replay/import/admin tooling that can submit stored object data

## 4.5 Trigger-relevant state heuristics
Pay attention to:
- custom `readObject`
- `readResolve`
- `readExternal`
- object fields consumed immediately after restore
- file, URL, command, template, or callback properties

---

# 5. False-Positive Controls

Do not mark a Java source as high-priority if:
- the value is constructed only by trusted server-side code,
- the mapper target is a strict DTO with no polymorphic type restoration,
- decoded data is only parsed into primitives or safe data structures,
- the source is integrity-verified and type-constrained before restore,
- the value does not reach deserialization-relevant code.

Use `Suspected source` or `Not enough evidence` if:
- helper wrappers hide the real deserializer,
- the active mapper configuration is not visible,
- cache or queue producer trust is unclear,
- stored blob write paths are missing,
- type allowlist behavior may exist elsewhere.

---

# 6. What Good Evidence Looks Like

Good Java source evidence includes:
- route, consumer, import handler, or helper entry point,
- source annotation or API such as `@RequestBody`, multipart file, cookie, cache read, or queue receive,
- propagation such as base64 decode, decompress, or `ByteArrayInputStream`,
- mapper or serializer helper receiving the value,
- type metadata or stored blob origin when visible.

Good source evidence answers:
1. Which Java entry point receives the serialized or restore-relevant value?
2. Is the value client-controlled, external-system-controlled, stored attacker-influenced, or trusted?
3. Which object restoration API or mapper should be audited next?
4. Is type metadata or trigger-relevant object state influenced?

---

# 7. Quick Java Source Checklist

- Are request bodies, uploads, cookies, or headers converted to streams or bytes?
- Are base64/compressed values decoded before a restore helper?
- Are `@type`, class names, or polymorphic fields accepted from input?
- Are cache/session/Redis/database blobs restored later?
- Are queue or RPC payloads decoded into objects?
- Are XMLDecoder, ObjectInputStream, Jackson/Fastjson, Hessian, Kryo, or custom decoder inputs traceable?
- Are restored fields file paths, URLs, commands, templates, or callbacks?
