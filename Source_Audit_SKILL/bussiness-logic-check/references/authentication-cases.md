# Authentication and Verification Logic Source Cases

## Purpose

This file defines authentication-flow and verification-flow source points for business logic source discovery.

Use it when the target workflow involves:
- login challenge flows
- OTP / SMS / email verification
- password reset
- account recovery
- account binding
- step-up verification
- device or contact confirmation

This reference is guidance, not proof. Always verify the real source origin, step order, challenge binding, expiry handling, and backend enforcement in code.

---

# 1. Authentication Source Discovery Points

Prioritize these source values and events:
- user ID, account ID, session ID, device ID
- phone number, email address, contact target, or binding target
- OTP code, reset token, challenge ID, verification ID
- purpose such as login, reset, bind, step-up, or recovery
- issued-at, expires-at, used-at, and attempt count
- resend, verify, reset, bind, and recover action names
- client-side verified flags or flow-step markers
- rate-limit keys for send and verify paths

Source questions:
- Which source identifies the account or contact target?
- Which source proves a challenge was issued and completed?
- Which source binds code/token to user, target, purpose, session, and time window?
- Which source limits send, resend, and verify attempts?
- Which source determines final beneficiary of reset or binding?

---

# 2. Authentication Source Patterns

## A-S1. Challenge artifact source
Example idea:
- request or database includes `challenge_id`, OTP code, reset token, or verification record.

Audit relevance:
Challenge artifacts decide whether later sensitive steps may run.

Follow-up:
- verify artifact is server-issued, purpose-bound, not expired, and not already used.

## A-S2. Target contact source
Example idea:
- phone or email comes from request input during send, verify, reset, or bind.

Audit relevance:
The target determines who receives verification and what account/contact is affected.

Follow-up:
- verify target is consistently bound from challenge issuance through final action.

## A-S3. Purpose and step source
Example idea:
- request includes `purpose=reset`, `type=bind`, or a client-side flow marker.

Audit relevance:
Purpose controls whether the same artifact can be reused across workflows.

Follow-up:
- verify purpose is stored server-side and matched during verification and final action.

## A-S4. Attempt and rate source
Example idea:
- limiter key, attempt counter, resend counter, IP/device/user/target key.

Audit relevance:
These sources determine brute-force and bombing resistance.

Follow-up:
- verify counters cover send and verify paths across user, target, device, and IP.

## A-S5. Beneficiary binding source
Example idea:
- final reset or binding endpoint accepts account ID, user ID, phone, or email.

Audit relevance:
The final beneficiary must match the verified challenge target.

Follow-up:
- verify the final action uses server-side challenge state rather than fresh request input alone.

---

# 3. What to Inspect in Code

Inspect:
- OTP issuance and verification logic
- reset token generation and invalidation
- challenge state persistence
- purpose binding
- expiry and used-at checks
- session, user, target, and device binding
- retry, resend, and verify attempt counters
- post-verification gates before reset, bind, or step-up action
- client-side verified flags or callback parameters
- rate limiting around send and verify paths

---

# 4. Source-Level Danger Signals

High-priority source signals:
- reset or bind target comes from final request instead of verified challenge record
- OTP lookup source is code only, without user/target/purpose
- client submits a `verified` flag or flow-step marker
- reset token source is not invalidated after success
- resend and verify paths use different limiter keys
- account ID and contact target are sourced independently late in the workflow

---

# 5. Case Templates

## Case Auth-S-1: Password reset source chain

Source focus:
Identify reset token or challenge ID, target account, purpose, expiry, used state, and final password-reset target.

Follow-up:
Verify the final reset action depends on server-side completed challenge state.

## Case Auth-S-2: OTP binding source

Source focus:
Identify OTP code, user, target phone/email, purpose, session, and time window.

Follow-up:
Verify lookup and verification bind all required dimensions.

## Case Auth-S-3: Verification replay source

Source focus:
Identify used-at, consumed flag, token ID, verification state, and retry path.

Follow-up:
Verify successful artifacts cannot be reused.

## Case Auth-S-4: Account binding beneficiary source

Source focus:
Identify who initiates binding, which contact is verified, and which account receives the binding.

Follow-up:
Verify actor, verified target, and beneficiary account remain consistent.
