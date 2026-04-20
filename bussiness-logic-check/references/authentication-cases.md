# Authentication and Verification Logic Cases

## Purpose

This file defines authentication-flow and verification-flow abuse patterns, code review targets, anti-patterns, and audit cases.

Use it when the target workflow involves:
- login challenge flows
- OTP / SMS / email verification
- password reset
- account recovery
- account binding
- step-up verification
- device or contact confirmation

This reference is guidance, not proof. Always verify the real step order, challenge binding, expiry handling, and backend enforcement in code.

---

# 1. Typical Authentication Workflow Rules

Common objects:
- verification challenge
- OTP code
- reset token
- session state
- login attempt
- binding request
- verified contact record

Typical rules:
- challenge must be issued before it can be consumed
- code must match the right user / target / purpose
- expired or used tokens must not be reusable
- a reset should require a valid completed challenge
- binding should attach only to the verified target
- step-up actions should not be reachable without prior success

---

# 2. Authentication Abuse Patterns

## A1. Step skipping
Risk:
Later-stage actions can be called directly without successful completion of earlier steps.

Look for:
- reset endpoint callable without challenge state
- binding confirm endpoint not tied to issued challenge
- verify-success flag inferred from client state only

## A2. Weak challenge binding
Risk:
Code or token is valid, but not bound tightly enough to:
- user
- target phone/email
- purpose
- session
- time window

Look for:
- OTP verified only by code value
- missing target binding
- same token usable for multiple purposes

## A3. Reusable or replayable verification artifacts
Risk:
OTP, reset token, or verification state can be reused.

Look for:
- missing one-time-use flag
- no used-at update
- no invalidation after success
- retry path reuses old token

## A4. Login or reset logic detached from rate control
Risk:
Challenge send or challenge verify paths can be spammed or brute-forced.

Look for:
- no per-user / per-target / per-IP rate checks
- no attempt counter
- no cooldown
- no lockout or backoff

## A5. Account-binding confusion
Risk:
Verified contact, account identity, and binding target become inconsistent.

Look for:
- phone verified for one session but bound to another account
- invite / bind / recover flow allows target swap
- beneficiary of verification differs from initiator

---

# 3. What to Inspect in Code

Inspect:
- OTP issuance and verification logic
- reset token generation and invalidation
- challenge state persistence
- purpose binding (reset vs login vs bind)
- expiry checks
- one-time-use updates
- session/user/target binding
- retry and resend logic
- post-verification gating before sensitive action
- rate limiting around send and verify paths

---

# 4. Code-Level Danger Signals

High-risk signals:
- `resetPassword()` callable without verified challenge lookup
- OTP lookup by code only
- token checked but not invalidated
- verified flag stored in client-controlled state
- bind action takes target from current request instead of verified challenge record
- resend and verify lack shared abuse controls

---

# 5. Case Templates

## Case Auth-1: Password reset step skipping

Expected rule:
Reset should require a valid, purpose-bound, not-yet-used verification artifact.

Audit focus:
Verify whether reset endpoints actually require server-side verified challenge state.

## Case Auth-2: OTP not bound to target or purpose

Expected rule:
OTP must be tied to the correct user/contact/purpose combination.

Audit focus:
Verify lookup keys and post-verification binding.

## Case Auth-3: Reusable verification token

Expected rule:
A successful token or code should not be reusable.

Audit focus:
Verify invalidation and one-time-use enforcement.

## Case Auth-4: Account binding beneficiary mismatch

Expected rule:
The verified identity and the final bound account should remain consistent.

Audit focus:
Verify who initiates, who verifies, and who ultimately benefits from the binding.
