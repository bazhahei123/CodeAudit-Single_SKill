# Promotion, Coupon, Reward, and Referral Cases

## Purpose

This file defines promotion-specific business abuse patterns, code review targets, anti-patterns, and audit cases.

Use it when the target workflow involves:
- coupon claiming
- coupon redemption
- discount calculation
- referral and invite rewards
- first-order / first-user campaigns
- loyalty points
- sign-up bonuses
- activity qualification checks
- redemption limits or eligibility logic

This reference is guidance, not proof. Always verify the real eligibility rules, redemption constraints, state changes, and beneficiary binding in code.

---

# 1. Typical Promotion Objects and Rules

Common objects:
- coupon
- coupon template
- redemption record
- promotion rule
- eligibility record
- invite relationship
- reward record
- points balance
- first-order / first-user marker

Typical rules:
- one user should not claim or redeem the same entitlement more than allowed
- one coupon should not be redeemable more than once
- discount stacking must follow explicit policy
- referral rewards should go only to valid inviter / invitee pairs
- "new user" or "first order" status should be derived server-side
- rewards should not be repeatedly issued under retries, race conditions, or alternate paths

---

# 2. Promotion Abuse Patterns

## PR1. Repeated claiming or redemption
Risk:
A user can claim or redeem the same benefit more times than intended.

Look for:
- missing unique claim/redeem record
- no terminal-state update
- no transaction or lock around claim path
- race between eligibility check and reward issuance

## PR2. Discount stacking or policy bypass
Risk:
Multiple discounts, promotions, or rewards are combined in ways the business did not intend.

Look for:
- independent discount application with no final conflict check
- multiple coupons accepted without explicit stacking rules
- server trusts client-provided discount result
- inconsistent rules between checkout, settlement, and refund logic

## PR3. Fake new-user / first-order qualification
Risk:
A user can appear eligible for new-user, first-order, or first-use incentives incorrectly.

Look for:
- qualification derived from request input
- incomplete server-side history check
- account / device / contact / tenant boundaries checked inconsistently
- alternate purchase path does not update qualification markers

## PR4. Referral or invite self-abuse
Risk:
A user can create cycles, self-referrals, duplicate referral credit, or reward the wrong beneficiary.

Look for:
- inviter and invitee not strongly separated
- bind / register / reward paths using different identity keys
- no anti-self-referral check
- no one-time reward issuance guard

## PR5. Reward replay or duplicate issuance
Risk:
Callback, retry, or repeated state transition issues the same points, coupon, or reward multiple times.

Look for:
- reward issuance without idempotency
- no processed-event table
- repeated async jobs can grant reward again
- same condition checked in multiple paths with no shared dedupe

## PR6. Redemption target mismatch
Risk:
The benefit is bound to one actor or object but applied to another.

Look for:
- coupon owner differs from checkout beneficiary
- inviter verified in one place but reward lands elsewhere
- cart/order/user identity not consistently bound
- delegated or merged-account flows with weak binding

---

# 3. What to Inspect in Code

Inspect:
- coupon claim / redeem service methods
- promotion eligibility calculation
- first-order / new-user detection logic
- referral relationship creation and validation
- reward issuance code
- points / coupon / voucher state changes
- checkout discount recomputation
- async reward jobs and retries
- dedupe / idempotency tables
- transaction boundaries and uniqueness constraints

---

# 4. Code-Level Danger Signals

High-risk signals:
- claim and redeem path checks eligibility before write, but without transaction or unique constraint
- discount total accepted from client or frontend-calculated payload
- first-order flag stored loosely and not recomputed from authoritative history
- inviter / invitee relation created before identity is finalized
- reward issued in both immediate path and async confirmation path
- promotion constraints checked in one order flow but not in alternate checkout path

---

# 5. Case Templates

## Case Promo-1: Duplicate coupon claim

Expected rule:
A user should not claim the same coupon or reward more than allowed.

Audit focus:
Verify uniqueness constraints, state transitions, and transactional protection around claim paths.

## Case Promo-2: Coupon stacking bypass

Expected rule:
Only explicitly allowed discount combinations should apply.

Audit focus:
Verify final server-side reconciliation of all discounts before order placement or payment.

## Case Promo-3: Self-referral or referral cycle abuse

Expected rule:
Referral reward should only apply to valid, distinct inviter / invitee pairs.

Audit focus:
Verify identity binding, anti-self-referral checks, and one-time reward issuance.

## Case Promo-4: Fake first-order qualification

Expected rule:
First-order or new-user status should be computed from authoritative server-side history.

Audit focus:
Verify how qualification is derived and updated across all purchase paths.

## Case Promo-5: Duplicate reward on retry

Expected rule:
The same qualifying event should not issue the same reward more than once.

Audit focus:
Verify idempotency and dedupe in reward issuance and async processing paths.
