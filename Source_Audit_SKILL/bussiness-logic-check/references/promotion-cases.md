# Promotion, Coupon, Reward, and Referral Source Cases

## Purpose

This file defines promotion-specific source points for business logic source discovery.

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

This reference is guidance, not proof. Always verify the real source origin, eligibility rules, redemption constraints, state changes, and beneficiary binding in code.

---

# 1. Promotion Source Discovery Points

Prioritize these source values and events:
- coupon code, coupon ID, template ID, promotion ID
- user ID, redeemer ID, coupon owner, account ID, tenant ID
- order ID, cart ID, checkout target, redemption target
- discount value, stacking list, promotion result, reward value
- eligibility marker such as new user, first order, activity complete, or membership level
- inviter ID, invitee ID, referral code, referral relationship
- reward event ID, claim record, redemption record, points ledger entry
- retry, callback, async reward, or duplicate issuance trigger

Source questions:
- Which source identifies the benefit or promotion rule?
- Which source identifies the actor, redeemer, inviter, invitee, and beneficiary?
- Which source determines eligibility and first-use status?
- Which source determines discount value or stacking behavior?
- Which source prevents duplicate claim, redemption, or reward issuance?

---

# 2. Promotion Source Patterns

## PR-S1. Coupon or promotion identifier source
Example idea:
- request includes coupon code, coupon ID, template ID, or promotion ID.

Audit relevance:
This source selects the promotion rule and benefit.

Follow-up:
- verify rule lookup, ownership, eligibility, usage count, and redemption state.

## PR-S2. Eligibility source
Example idea:
- new-user flag, first-order marker, membership level, or activity qualification.

Audit relevance:
Eligibility sources determine whether a benefit may be claimed or applied.

Follow-up:
- verify eligibility is derived from authoritative server-side history.

## PR-S3. Discount and stacking source
Example idea:
- request includes discount amount, coupon list, stacking flag, or frontend-calculated promotion result.

Audit relevance:
Discount sources influence value and accounting integrity.

Follow-up:
- verify final server-side reconciliation and explicit stacking policy.

## PR-S4. Referral relationship source
Example idea:
- referral code, inviter ID, invitee ID, registration source, or invite binding.

Audit relevance:
Referral sources determine who receives a reward and whether self-abuse is possible.

Follow-up:
- verify anti-self-referral, cycle prevention, and one-time reward issuance.

## PR-S5. Reward issuance source
Example idea:
- qualifying event, async job, callback, reward event ID, or claim record triggers points/coupon issuance.

Audit relevance:
Reward sources determine whether retries or alternate paths duplicate benefits.

Follow-up:
- verify idempotency, dedupe records, and transactional claim/redeem behavior.

---

# 3. What to Inspect in Code

Inspect:
- coupon claim and redeem service methods
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

# 4. Source-Level Danger Signals

High-priority source signals:
- new-user or first-order marker comes from request input
- coupon discount result is accepted from frontend payload
- coupon owner and checkout beneficiary are sourced separately
- inviter and invitee source values are not strongly separated
- claim/redeem source lacks unique record or state marker
- reward is triggered from both immediate and async paths
- eligibility source differs across checkout, payment, and refund paths

---

# 5. Case Templates

## Case Promo-S-1: Duplicate claim source

Source focus:
Identify coupon, user, claim record, redemption record, and uniqueness source.

Follow-up:
Verify one-time or limited-use constraints and transactional protection.

## Case Promo-S-2: Discount stacking source

Source focus:
Identify all coupon/promotion sources and final discount value source.

Follow-up:
Verify server-side stacking policy and final reconciliation.

## Case Promo-S-3: Referral beneficiary source

Source focus:
Identify inviter, invitee, referral code, account binding, and reward owner.

Follow-up:
Verify actor/beneficiary consistency and anti-self-referral logic.

## Case Promo-S-4: First-order qualification source

Source focus:
Identify first-order/new-user marker and authoritative history source.

Follow-up:
Verify qualification is not client-controlled and is updated across all purchase paths.

## Case Promo-S-5: Reward retry source

Source focus:
Identify qualifying event ID, reward issuance trigger, retry job, and processed reward record.

Follow-up:
Verify duplicate reward issuance is prevented.
