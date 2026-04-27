# Path Traversal Common Cases

## Purpose

This file defines shared path traversal, arbitrary file access, unsafe file path handling, archive extraction traversal, and local resource inclusion concepts, audit logic, anti-patterns, false-positive controls, and finding standards that apply across languages and frameworks.

Use this file as the base reference for path traversal review before loading any language-specific reference.

This file explains:
- what path traversal and unsafe path handling are,
- what counts as a source, path-construction point, sink, and control,
- how to reason about read, write, delete, overwrite, include/load, and extraction contexts,
- why normalization alone is not the same as safe containment,
- when to report `Confirmed`, `Suspected`, or `Not enough evidence`.

This reference is guidance, not proof. Do not report a vulnerability only because code resembles a pattern described here. Always verify the real data flow, real path handling, and real file operation in the target code.

---

# 1. Core Concepts

## 1.1 What path traversal risk is
Path traversal risk exists when attacker-controlled or weakly trusted input can influence a file path, resource path, extraction path, or include/load path in a way that escapes the intended scope of filesystem or local resource access.

The core question is:

**Can attacker-controlled input change what local file or resource is read, written, deleted, overwritten, extracted, or included beyond the intended safe scope?**

## 1.2 Source, propagation, path construction, and sink

### Source
A source is any attacker-controllable input, including:
- query parameters
- path parameters
- request body fields
- headers
- cookies
- uploaded filenames
- archive entry names
- stored path values
- config values influenced by external actors
- queue payloads
- API responses derived from user-controlled content

### Propagation
Propagation means the input is copied, decoded, normalized, concatenated, joined, stored, or transformed before reaching a path-construction point or file sink.

Do not assume a renamed or "cleaned" variable is safe unless the constraint is actually enforceable and visible.

### Path construction
A path-construction point is where the application turns a value into a filesystem or local resource path.

Examples:
- base directory join
- string concatenation
- path normalization
- archive entry destination assembly
- template/resource path selection

### Sink
A sink is the place where the path is used in a meaningful file or local resource operation, including:
- read
- write
- delete
- overwrite
- move / rename
- copy
- extract
- include / require
- template load
- local resource load

---

# 2. Shared Attack Surfaces

Prioritize these attack surfaces first:

- file download and preview endpoints
- upload destination selection
- import and export handlers
- log, report, and config readers
- file move / rename / copy / delete handlers
- archive extraction code
- template and local resource loaders
- temporary-file and cleanup logic
- admin file-management tools
- background jobs processing stored paths
- legacy helper methods and alternate file-operation paths

---

# 3. Shared Risk Categories

## 3.1 Arbitrary file read
User input influences a path used for file reading, preview, download, import, or local load.

Examples:
- download handler
- log viewer
- config reader
- report preview
- local file include-like resource load

## 3.2 Arbitrary file write / overwrite
User input influences a path used for writing or overwriting files.

Examples:
- export destination
- upload save destination
- generated report path
- temporary file path
- cache file path

## 3.3 Arbitrary file delete / move / rename
User input influences a path used for destructive or relocation operations.

Examples:
- delete file endpoint
- cleanup logic
- rename/move handler
- archive extraction overwrite

## 3.4 Archive extraction traversal
Archive entry names influence extraction destinations outside the intended root.

Examples:
- zip slip
- tar traversal
- nested path escape in import bundles

## 3.5 Local include / resource load abuse
User input influences:
- template name
- include path
- theme/resource selector
- local file loader

Why important:
Even if the operation is not a normal "download", it can still create local resource disclosure or unsafe include behavior.

---

# 4. Normalization vs Canonicalization

## 4.1 Normalization
Normalization typically removes redundant path syntax.

Examples:
- collapsing `.` or repeated separators
- simplifying some relative path syntax

Why it is not enough:
Normalization alone does not prove the final path remains inside the intended safe base directory.

## 4.2 Canonicalization / real path resolution
Canonicalization attempts to resolve the actual effective path.

Examples:
- resolving symlinks
- deriving final filesystem location
- generating a canonical absolute path

Why it matters:
Safe containment checks usually need to compare the **resolved** target path against an allowed base path.

## 4.3 Common mistake
A very common mistake is:
- normalize or strip `../`
- do a simple prefix or substring check
- trust the result without real containment validation

This is often insufficient.

---

# 5. Base Directory Containment

## 5.1 What strong containment usually looks like
Strong protections typically include:
- fixed base directory
- server-generated filenames when possible
- canonical path resolution
- comparison against resolved allowed base
- strict allowlist for allowed filenames or resource keys
- refusing absolute paths or unexpected separators
- safe extraction root enforcement for archives

## 5.2 What does not automatically mean the code is safe

The following do **not** automatically mean the code is safe:
- calling `normalize()` only
- replacing `../` only
- checking `startsWith(base)` before canonicalization
- blocking only one separator style
- checking for `".."` only
- trusting upload filename directly
- trusting archive entry names directly

## 5.3 Symlink and check/use concerns
Even a containment check may be weak if:
- symlinks are followed unexpectedly
- the check is done before the final path changes
- the code validates one string but opens another resolved path later
- different helper layers apply different path rules

---

# 6. Archive Extraction Safety

Archive extraction needs its own review.

## 6.1 Risk model
Each archive entry name can behave like attacker-controlled path input.

## 6.2 Safe expectations
Safe extraction typically requires:
- validating each entry path
- resolving destination relative to a fixed extraction root
- rejecting entries that escape the extraction root
- handling overwrite behavior safely
- applying the same rules to nested directories and special paths

## 6.3 Common mistake
A common mistake is:
- `dest = base + "/" + entryName`
- write extracted file directly
without validating the resolved final path.

---

# 7. Shared Anti-Patterns

These are common danger signals across languages.

## A1. Concatenated file path
High-risk pattern:
- base directory string + user input
- manual path string assembly
- direct use of request parameter as path

Why risky:
The attacker may escape intended scope.

## A2. Unsafe join without containment check
High-risk pattern:
- join base path and user path
- no canonical containment validation afterwards

Why risky:
Joining a base path does not automatically keep the target inside the base.

## A3. String blacklist filtering only
High-risk pattern:
- replace `"../"`
- reject only obvious traversal strings
- block only one separator style

Why risky:
Path parsing and representation are more complex than a simple substring check.

## A4. Trusting uploaded filename or stored path
High-risk pattern:
- save or read using attacker-controlled filename directly
- trust previously stored path as safe

Why risky:
Storage does not automatically make a path trustworthy.

## A5. Read/write/delete mismatch
High-risk pattern:
- one operation is constrained safely
- another equivalent operation is not

Examples:
- preview safe, download unsafe
- upload safe, overwrite path unsafe
- normal delete safe, cleanup helper unsafe

## A6. Archive entry trust
High-risk pattern:
- extraction path assembled from entry names directly
- no root-containment validation for archive entries

Why risky:
Archive metadata behaves like attacker-controlled path input.

## A7. Include/resource path trust
High-risk pattern:
- template include by user-selected name
- local resource loader trusts request path or theme path

Why risky:
Local file/resource access may escape intended allowed resources.

## A8. False confidence from one helper
High-risk pattern:
- one helper claims to sanitize the path
- other code paths bypass it
- helper normalizes but does not truly enforce containment

Why risky:
Wrapper presence alone does not prove safety.

---

# 8. False-Positive Controls

Do not report a vulnerability as `Confirmed` if:
- the path value is provably constant or server-controlled,
- the target path is derived from a strict allowlist of safe resource keys,
- the final resolved path is clearly constrained inside a trusted base directory,
- the apparent sink is unreachable from attacker-controlled input,
- archive entries are clearly validated against extraction-root containment before write,
- local resource selection uses only fixed template/resource identifiers, not raw paths.

Use `Suspected` or `Not enough evidence` if:
- a risky sink exists but the true canonicalization or allowlist logic may be hidden elsewhere,
- a path is user-influenced but the actual final resolved path is not visible,
- a wrapper helper may enforce containment but its implementation is unavailable,
- file operation context is visible but the source-to-sink path is incomplete.

Do not over-claim based only on:
- the presence of file APIs,
- the presence of path joins,
- the presence of `../` in test-looking strings,
- the absence of one visible check if a shared helper may exist,
- an archive extraction feature without visible untrusted entry control.

---

# 9. Finding Classification

## Confirmed
Use `Confirmed` when there is clear evidence that:
- attacker-controlled or weakly trusted input reaches a path construction point or file sink,
- protection is absent, weak, or bypassable,
- and out-of-scope file or resource access is plausibly achievable.

## Suspected
Use `Suspected` when:
- there is a high-risk source-to-sink pattern,
- the path looks attacker-influenced,
- but final containment, canonicalization, or allowlist behavior remains hidden.

## Not enough evidence
Use `Not enough evidence` when:
- the sink is visible but the source path is unclear,
- the source is visible but the final path handling is hidden,
- or wrapper / framework path behavior cannot be verified from available code.

## Probably safe
Use `Probably safe` when:
- the path flow is visible,
- resolved path containment or strict allowlisting is clearly enforced,
- and no alternate weaker path to the same operation is evident.

---

# 10. What Good Evidence Looks Like

Strong path traversal findings usually include:
- the exact entry point
- the tainted input source
- the path construction point
- the file or resource sink
- the file operation context
- the missing or weak containment / canonicalization / allowlist control
- the reason attacker input can escape intended scope

Good findings usually answer:
1. What input is attacker-controlled?
2. Where does it become a path?
3. What file or resource operation uses it?
4. What protection should have constrained it?
5. Why is out-of-scope access plausible?

---

# 11. Shared Remediation Guidance

Preferred fixes include:
- keep file operations inside fixed base directories
- resolve and compare canonical paths before use
- use strict allowlists for filenames, resource keys, and template identifiers
- generate server-side filenames instead of trusting user input
- validate archive entries before extraction
- treat stored paths as untrusted unless revalidated
- keep read, write, delete, and extract paths under the same containment rules

Avoid weak fixes such as:
- replacing `"../"` only
- blocking one separator style only
- normalization without real containment checks
- trusting upload filenames directly
- validating only one path while using another path later

---

# 12. Shared Quick Checklist

Use this as a reminder, not as a substitute for reasoning.

- Is there a real attacker-controlled or weakly trusted source?
- Does it reach a path construction point?
- What file or local resource sink uses the path?
- Is the operation read, write, delete, overwrite, extract, or include/load?
- Is there true resolved-path containment?
- Are symlink and alternate path representations handled safely?
- Are archive entries validated before extraction?
- Are alternate paths to the same file operation equally constrained?
