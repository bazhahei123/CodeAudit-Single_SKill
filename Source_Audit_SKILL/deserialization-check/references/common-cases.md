# Unsafe Deserialization Source Common Cases

## Purpose

This file defines shared unsafe deserialization source concepts, source classification logic, propagation patterns, false-positive controls, and evidence standards that apply across languages and frameworks.

Use this file as the base reference for unsafe deserialization source review before loading any language-specific reference.

This file explains:
- what a deserialization source point is,
- how to distinguish source, propagation, sink, trigger, and gadget from a source-review perspective,
- how to classify trust boundaries,
- how to reason about direct and second-order source paths,
- when to record `Confirmed source`, `Suspected source`, `Not enough evidence`, or `Probably irrelevant`.

This reference is guidance, not proof. Do not report a vulnerability only because code contains a source described here. Always verify the real source origin, real propagation path, and real downstream deserialization relevance in the target code.

---

# 1. Core Concepts

## 1.1 Deserialization source point

A deserialization source point is where data that may later be restored into objects, types, or structured runtime state enters or becomes available to the backend code path.

Examples:
- request body bytes
- uploaded file content
- cookie or session blob
- cache value
- queue message body
- saved template or metadata
- base64-encoded serialized data
- polymorphic type field
- YAML object tag
- internal RPC payload

A source point is an audit starting point, not proof of a vulnerability.

## 1.2 Source, propagation, sink, trigger, and gadget

### Source
The origin of data that may be decoded, loaded, mapped, restored, or passed to object reconstruction.

Examples:
- `payload` from request body
- `session` from cookie
- `blob` from cache
- `message.body` from queue
- `@type` from JSON
- uploaded archive metadata

### Propagation
The way a source is copied, encoded, decoded, compressed, encrypted, signed, wrapped, stored, normalized, or transformed before it reaches a sink.

Examples:
- base64 decode before restore
- gzip decompress before object load
- JSON wrapper carrying binary object data
- stored database field later read by worker
- helper method hiding the real restore API

### Sink
The place where the application restores objects, types, or structured runtime state.

Examples:
- native object deserializers
- unsafe YAML/XML loaders
- polymorphic object mappers
- cache/session restore helpers
- binary protocol decoders
- queue consumers that decode object payloads

### Trigger
The mechanism by which dangerous behavior may occur during or after restoration.

Examples:
- magic methods
- lifecycle hooks
- restore callbacks
- implicit property access
- post-restore business logic trusting object state

### Gadget
A class, method path, or behavior chain that may be abused after unsafe restoration.

Source discovery does not need to prove a gadget. It should identify source values that deserve follow-up sink and trigger review.

---

# 2. Trust Boundary Classification

## 2.1 Client-controlled
The value comes from request path, query string, request body, form data, headers, cookies, GraphQL variables, RPC arguments, uploaded files, or user-supplied import rows.

## 2.2 External-system-controlled
The value comes from a queue message, webhook, partner event, internal service response, provider payload, RPC frame, cache replication event, or integration sync. It is only trusted after origin, integrity, and scope are verified.

## 2.3 Stored attacker-influenced
The value is read from database, cache, session store, object storage, file metadata, saved templates, or admin/import/replay storage, and earlier attacker influence is possible.

## 2.4 Server-trusted
The value is constructed by trusted server-side code, loaded from trusted configuration, verified with strong integrity protection, or derived from a trusted framework context with no attacker-controlled content.

## 2.5 Mixed
The value combines untrusted input with trusted data, or an untrusted value is verified, constrained, or transformed before use.

## 2.6 Unclear
The origin cannot be determined from visible code.

---

# 3. Shared Source Categories

## 3.1 Direct serialized payload sources
Values that may reach a restore sink in the same request or processing path.

Examples:
- request body
- file upload content
- cookie blob
- header token
- form field carrying serialized data

## 3.2 Encoded or wrapped sources
Values that look transformed before restoration.

Examples:
- base64 payload
- compressed bytes
- encrypted-looking blob
- signed-looking cookie
- JSON field carrying serialized bytes
- XML/YAML text with object tags

## 3.3 Stored and second-order sources
Values written earlier and restored later.

Examples:
- saved filter
- saved template
- cached object blob
- session store entry
- queue payload
- stored metadata
- uploaded file later processed by worker

## 3.4 Type and object selection sources
Values that influence the class, object type, constructor, mapper target, or reconstruction path.

Examples:
- `@type`
- `class`
- `type`
- YAML object tag
- Java polymorphic metadata
- PHP serialized class name
- Python pickle reduction data

## 3.5 Trigger-relevant state sources
Values that become object properties or nested state after restoration and may influence magic methods, lifecycle hooks, callbacks, or post-restore logic.

Examples:
- file path property
- URL property
- command argument property
- callback name
- template path
- object graph relationship

---

# 4. Direct vs Second-Order Source Paths

## 4.1 Direct source path
The source reaches a deserialization sink in the same request or immediate processing path.

Examples:
- request body passed into object deserializer
- uploaded file parsed directly into objects
- cookie decoded and restored immediately

## 4.2 Second-order source path
The source is stored first, then restored in another code path.

Examples:
- saved metadata later loaded by worker
- cache value later restored by service
- queue payload produced from user-controlled data
- admin replay tool processes previously uploaded content

Second-order source points should include both:
- where the value is written or influenced,
- where the value is later restored or decoded, when visible.

---

# 5. Shared Source Patterns

## S1. Request body serialized payload
Example idea:
- raw request bytes, form field, JSON field, or multipart field is passed toward a decoder or restore helper.

Audit relevance:
Direct attacker-controlled payloads are high-priority source points for deserialization review.

## S2. Cookie or session blob
Example idea:
- cookie value is decoded, parsed, or restored into session state or objects.

Audit relevance:
Encoding alone is not integrity. Verify whether strong signing/MAC and safe restore behavior exist.

## S3. Uploaded file or metadata
Example idea:
- import file, archive metadata, image metadata, or user-supplied file path is later processed by object loaders or metadata handlers.

Audit relevance:
File content and metadata are common direct or second-order source points.

## S4. Queue, cache, or internal RPC payload
Example idea:
- worker consumes `message.body`, cache blob, or internal protocol frame and restores it.

Audit relevance:
Internal transport does not automatically make data trusted.

## S5. Type metadata or object tag
Example idea:
- source contains `@type`, class name, YAML tag, or serialized object class.

Audit relevance:
Type selection can determine what object behavior becomes reachable.

## S6. Stored blob later restored
Example idea:
- database or cache stores attacker-influenced serialized content that a later process restores.

Audit relevance:
Second-order source paths often hide the original attacker influence.

---

# 6. False-Positive Controls

Do not record a source point as high-priority if:
- the value is constant or constructed only by trusted server-side code,
- the value is parsed only into primitives or strict DTOs with no object/type restoration relevance,
- the downstream deserialization relevance is absent,
- strong integrity validation clearly happens before any unsafe restore behavior,
- the source cannot be influenced by an attacker or weakly trusted producer.

Use `Suspected source` or `Not enough evidence` if:
- the source is visible but downstream restore behavior is hidden,
- the sink is visible but input origin is hidden,
- a helper wrapper may verify integrity or enforce safe types,
- stored data may or may not be attacker-influenced,
- framework configuration may constrain type restoration but is not visible.

Do not over-claim based only on:
- serializer or parser API names,
- JSON/YAML/XML/binary format names,
- base64 or compression,
- stored data existence,
- magic methods or lifecycle hooks without source connection.

---

# 7. Source Classification

## Confirmed source
Use `Confirmed source` when there is clear evidence that:
- the source origin is visible,
- it reaches deserialization-relevant code or a likely restore path,
- and the trust boundary can be classified.

## Suspected source
Use `Suspected source` when:
- a value appears deserialization-relevant,
- the origin or downstream restore use is partially visible,
- but hidden framework, helper, integrity, allowlist, or storage behavior may change the classification.

## Not enough evidence
Use `Not enough evidence` when:
- the source origin cannot be determined,
- the downstream restore use cannot be determined,
- or critical framework/deserializer behavior is not visible.

## Probably irrelevant
Use `Probably irrelevant` when:
- the value is visible,
- downstream use is visible,
- and it does not influence object restoration, type restoration, unsafe loader behavior, stored serialized state, or trigger-relevant object state.

---

# 8. What Good Evidence Looks Like

Strong source findings usually include:
- the exact entry point
- the source value or payload name
- the source origin and trust boundary
- propagation steps such as decode, decompress, unwrap, or storage
- the first downstream deserialization-relevant use
- type metadata, stored blob, or trigger-relevant state if visible
- the follow-up sink/control check needed

Good source points usually answer:
1. Where does the serialized or restore-relevant value enter?
2. Who controls or can influence it?
3. Is it direct, encoded, stored, external, or second-order?
4. What deserialization or object-restoration behavior should be checked next?
5. What code proves the connection?

---

# 9. Shared Follow-up Guidance

After source discovery, the unsafe deserialization audit should verify:
- whether the source reaches a real object-restoration sink,
- whether only safe primitives or strict DTOs are restored,
- whether type restoration is allowlisted and constrained,
- whether strong integrity and trusted-origin checks apply before restore,
- whether magic methods, lifecycle hooks, or gadget-like behavior are reachable,
- whether second-order stored data is attacker-influenced,
- whether alternate restore paths apply the same safety controls.

Avoid weak conclusions such as:
- source exists, therefore vulnerability exists,
- JSON is used, therefore it is safe,
- base64 is used, therefore it is protected,
- data is internal, therefore it is trusted,
- helper exists, therefore restore behavior is safe.

---

# 10. Shared Quick Checklist

Use this as a reminder, not as a substitute for reasoning.

- Is there a client-controlled or weakly trusted serialized payload source?
- Is the source encoded, compressed, wrapped, or stored before restore?
- Is any type metadata, class name, or object tag attacker-influenced?
- Can stored data become a second-order restore source?
- Are queue, cache, RPC, session, and cookie values treated as trusted without proof?
- Does restored object state include file paths, URLs, command arguments, callbacks, or other trigger-relevant properties?
- What sink and control should review this source next?
