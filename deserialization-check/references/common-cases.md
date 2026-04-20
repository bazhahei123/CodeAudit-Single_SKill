# Unsafe Deserialization Common Cases

## Purpose

This file defines shared unsafe deserialization concepts, audit logic, anti-patterns, false-positive controls, and finding standards that apply across languages and frameworks.

Use this file as the base reference for unsafe deserialization review before loading any language-specific reference.

This file explains:
- what unsafe deserialization is,
- what counts as a source, sink, trigger, gadget, and control,
- how to distinguish direct and second-order deserialization risks,
- why trust boundaries and integrity protection matter,
- when to report `Confirmed`, `Suspected`, or `Not enough evidence`.

This reference is guidance, not proof. Do not report a vulnerability only because code resembles a pattern described here. Always verify the real data flow, real deserialization sink, and real dangerous post-restore behavior in the target code.

---

# 1. Core Concepts

## 1.1 What unsafe deserialization is
Unsafe deserialization occurs when untrusted or weakly trusted data is restored into objects, types, or structured runtime state in a way that can trigger dangerous behavior, violate integrity boundaries, or enable downstream exploitation.

The core question is:

**Can attacker-controlled data influence object restoration, type restoration, lifecycle behavior, or post-deserialization execution in a dangerous way?**

## 1.2 Source, propagation, sink, trigger, and gadget

### Source
A source is any attacker-controllable input, including:
- query parameters
- path parameters
- request body fields
- headers
- cookies
- uploaded files or metadata
- session blobs
- cache values
- queue messages
- stored values previously written by a user or admin
- internal service responses derived from attacker-controlled input

### Propagation
Propagation means the input is copied, encoded, compressed, signed, wrapped, stored, normalized, or transformed before reaching a deserialization sink.

Do not assume a renamed or base64-decoded value is safe.

### Sink
A sink is the place where the application restores objects, types, or structured runtime state, including:
- native deserialization APIs
- object mappers
- unsafe loaders
- framework serializer helpers
- binary decoders
- cache/session restoration helpers
- import handlers that rebuild objects
- message consumers that decode object payloads

### Trigger
A trigger is the mechanism by which dangerous behavior occurs during or after deserialization.

Examples:
- magic methods
- lifecycle hooks
- custom restore callbacks
- implicit method invocation
- property access with side effects
- post-deserialization business logic that trusts the restored object

### Gadget
A gadget is a class, method path, or chain of behavior that can be abused after unsafe object restoration to cause impact.

Examples:
- command execution
- arbitrary file access
- SSRF
- authentication bypass
- privilege escalation
- object state corruption
- denial of service

---

# 2. Shared Unsafe Deserialization Attack Surfaces

Prioritize these attack surfaces first:

- request-body object parsing
- file import and metadata processing
- session and cookie restoration
- cache reloads
- message queue consumers
- background jobs
- internal RPC or binary protocol handlers
- saved templates or stored blobs later restored
- admin replay / import / debug tooling
- legacy serializer helpers
- framework serializer and object-mapper configuration

---

# 3. Direct vs Second-Order Deserialization

## 3.1 Direct deserialization
Attacker input reaches a deserialization sink in the same request or immediate processing path.

Examples:
- request body passed into object deserializer
- uploaded file parsed directly into objects
- cookie decoded and restored immediately

## 3.2 Second-order deserialization
Attacker input is stored first, then later restored in another code path.

Examples:
- saved filters or templates later decoded
- session-like blobs stored in cache and later restored
- queue payloads produced from attacker-controlled input
- metadata written to disk/database and later loaded as an object

Second-order cases matter because the dangerous sink may be separated in time and code path from the original source.

---

# 4. Shared Anti-Patterns

These are common danger signals across languages.

## A1. Deserialization of untrusted input
High-risk pattern:
- request data directly passed into deserialize / load / decode object API

Why risky:
The attacker directly controls the data entering object restoration.

## A2. Trusting integrity-free client data
High-risk pattern:
- client-controlled session blob
- cookie object state
- base64-encoded object without signature
- weak or absent MAC/integrity check

Why risky:
The application may treat attacker-modified serialized data as trusted internal state.

## A3. Unsafe loader or permissive type restoration
High-risk pattern:
- loader restores arbitrary objects
- polymorphic type resolution accepts attacker-chosen classes
- framework default restores more than plain primitives

Why risky:
The attacker may influence the type or behavior of restored objects, not just their data.

## A4. Dangerous lifecycle or magic-method behavior
High-risk pattern:
- auto-triggered lifecycle hooks
- post-restore callbacks
- magic methods with side effects
- property access or cleanup logic that performs dangerous operations

Why risky:
Dangerous behavior may occur without explicit business code calling the method directly.

## A5. Framework wrapper hides the sink
High-risk pattern:
- helper method hides underlying deserialize/load/object-map behavior
- service appears to restore a "safe" structure but internally allows objects or unsafe handlers

Why risky:
Unsafe deserialization may be missed if only direct API names are searched.

## A6. Stored attacker-controlled blob later restored
High-risk pattern:
- stored serialized value reused later
- admin/import tool replays stored object data
- background consumer restores data that was originally attacker-controlled

Why risky:
The exploit path may be delayed and overlooked.

## A7. Internal-only trust assumption
High-risk pattern:
- code assumes queue/cache/internal RPC data is trusted
- no validation or integrity check because the source is considered "internal"

Why risky:
An attacker may still influence the upstream producer or stored value.

## A8. Safe-looking format with unsafe object behavior
High-risk pattern:
- JSON, YAML, XML, or custom format assumed safe
- object mapper or loader still restores arbitrary or dangerous object types

Why risky:
The danger is often in object restoration behavior, not only in the transport format name.

---

# 5. Shared Protection Model

## 5.1 What strong protection usually looks like

Strong protections typically include:
- using non-object safe data formats for untrusted input
- restoring only primitives or strictly defined DTOs
- strict class or type allowlists
- trusted-origin guarantees
- strong integrity protection on serialized state
- disabling dangerous loader modes
- avoiding native object deserialization for attacker-influenced data
- separating untrusted input parsing from dangerous object reconstruction

## 5.2 What does not automatically mean the code is safe

The following do **not** automatically mean the code is safe:
- using JSON instead of a binary format
- using a framework serializer
- base64 encoding
- signing in one path but not verifying in another
- calling a helper with a harmless name
- data being stored in a database or cache
- data being produced by an internal service

## 5.3 Integrity and trust boundaries matter
A deserialization path may still be dangerous even if:
- the sink is not publicly reachable,
- the data is "internal",
- the payload is stored first and restored later,
- the data is encoded but not strongly integrity-protected.

---

# 6. False-Positive Controls

Do not report a vulnerability as `Confirmed` if:
- the input is provably constant or trusted-only,
- the deserializer restores only plain primitives or fixed safe structures,
- strong integrity protection and trusted-origin guarantees clearly apply,
- dangerous classes, callbacks, or type restoration are clearly impossible,
- the apparent sink is unreachable from attacker-controlled input.

Use `Suspected` or `Not enough evidence` if:
- the sink exists but attacker influence on the input is unclear,
- the input is attacker-controlled but the final restore behavior is hidden,
- integrity, allowlists, or safe loader settings may exist elsewhere but cannot be verified,
- a gadget trigger is plausible but not visible,
- the framework wrapper obscures the true behavior.

Do not over-claim based only on:
- the presence of a serializer or parser API,
- the use of JSON, YAML, or XML by name,
- the existence of magic methods alone,
- the existence of a class with side effects alone,
- the fact that data is stored before being reused.

---

# 7. Finding Classification

## Confirmed
Use `Confirmed` when there is clear evidence that:
- attacker-controlled or weakly trusted data reaches a deserialization sink,
- protection is absent, weak, or bypassable,
- and dangerous post-deserialization behavior or downstream impact is plausibly reachable.

## Suspected
Use `Suspected` when:
- there is a high-risk source-to-sink pattern,
- integrity or trusted-origin assumptions appear weak,
- or dangerous trigger behavior seems plausible,
- but one or more critical enforcement details remain hidden.

## Not enough evidence
Use `Not enough evidence` when:
- the sink is visible but trust control is not,
- the source is visible but the trigger path is not,
- or framework behavior cannot be verified from the available code.

## Probably safe
Use `Probably safe` when:
- the path is visible,
- only safe structures are restored,
- integrity or trust controls are clearly applied,
- and no dangerous trigger or class behavior is evident.

---

# 8. What Good Evidence Looks Like

Strong unsafe deserialization findings usually include:
- the exact entry point
- the tainted or weakly trusted input source
- the deserialization sink
- the restored type or object behavior
- the dangerous trigger or gadget path
- the missing integrity, allowlist, or trust-boundary control
- the reason attacker-controlled data can influence execution or object behavior

Good findings usually answer:
1. What input is attacker-controlled or weakly trusted?
2. Where is it restored?
3. What object or type behavior becomes dangerous?
4. What control should have stopped or constrained that restoration?
5. Why is exploitation plausible?

---

# 9. Shared Remediation Guidance

Preferred fixes include:
- avoid native object deserialization for untrusted input
- deserialize only into safe primitive structures or strict DTOs
- use strong class/type allowlists
- disable dangerous loader or polymorphic modes
- apply strong integrity protection to serialized state
- treat cache, queue, session, and stored blobs as untrusted unless verified
- redesign workflows to avoid restoring attacker-controlled objects

Avoid weak fixes such as:
- relying on encoding only
- trusting internal sources without verification
- relying on blacklist-only class filtering
- assuming storage implies trust
- assuming signed data is safe without verifying the full trust chain

---

# 10. Shared Quick Checklist

Use this as a reminder, not as a substitute for reasoning.

- Is there a real attacker-controlled or weakly trusted source?
- Does it reach a deserialization or object-restoration sink?
- Does the sink restore objects, types, or only safe primitives?
- Is strong integrity protection present and actually verified?
- Are dangerous classes, lifecycle hooks, or triggers reachable?
- Can stored data later become a second-order deserialization path?
- Are alternate restore paths equally constrained?
- Is the code relying on trust assumptions instead of enforceable controls?
