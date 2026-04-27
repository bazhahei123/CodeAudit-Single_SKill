# Workflow and Lifecycle Cases

## Purpose

This file defines workflow-specific business abuse patterns, code review targets, anti-patterns, and audit cases.

Use it when the target workflow involves:
- draft / review / approve / publish
- open / assign / resolve / close
- create / confirm / finalize / archive
- cancel / revert / resubmit
- moderation
- lifecycle transitions with role or state gating

This reference is guidance, not proof. Always verify the real state model, transition guards, approval rules, and alternate path handling in code.

---

# 1. Typical Workflow Rules

Typical rules:
- transitions must occur in allowed order
- only certain roles may perform specific transitions
- terminal states should block further mutation
- repeated approval, publish, cancel, or finalize actions should be blocked
- alternate operational paths should enforce the same transition rules
- dependent side effects should only happen after valid transition success

---

# 2. Workflow Abuse Patterns

## W1. Invalid state transition allowed
Risk:
Action is allowed even though the object is in the wrong state.

Look for:
- missing current-state check
- generic update method callable from multiple states
- terminal-state mutation

## W2. Step skipping
Risk:
Object reaches later workflow stage without completing required earlier phase.

Look for:
- publish without approval
- finalize without confirmation
- close without resolution
- archive without completion

## W3. Repeated terminal action
Risk:
Approve, publish, cancel, or finalize runs more than once.

Look for:
- no terminal-state guard
- no idempotent final-action protection
- repeated side effects on retry

## W4. Role and state rule split inconsistently
Risk:
One layer checks role, another checks state, but no path checks both reliably.

Look for:
- controller checks role only
- service checks state only
- alternate batch/admin path checks neither fully

## W5. Alternate path bypass
Risk:
Batch, import, retry, admin, or queue path applies weaker workflow rules than the primary path.

Look for:
- normal UI path guarded
- async/ops path missing same transition validation

---

# 3. What to Inspect in Code

Inspect:
- workflow service methods
- state transition checks
- role + state combined validation
- terminal-state protection
- retry behavior for final actions
- approval / publish / archive handlers
- batch/admin/ops paths
- queue or callback paths changing lifecycle state
- side effects tied to state changes

---

# 4. Code-Level Danger Signals

High-risk signals:
- action updates `status` directly with no current-state check
- approval and publish handled by separate endpoints with weak coupling
- terminal states not protected
- admin batch path bypasses normal workflow validator
- repeated finalization repeats downstream side effects

---

# 5. Case Templates

## Case WF-1: Publish before approval

Expected rule:
Publish should require successful review or approval state first.

Audit focus:
Verify server-side enforcement of prerequisite state.

## Case WF-2: Repeated approval or finalization

Expected rule:
Final actions should be one-time and idempotent.

Audit focus:
Verify terminal-state guard and side-effect dedupe.

## Case WF-3: Alternate admin path bypass

Expected rule:
Operational or batch paths should enforce the same lifecycle rules as the main path.

Audit focus:
Compare main route vs admin / retry / batch implementation.

## Case WF-4: Terminal-state mutation

Expected rule:
Archived, finalized, or closed objects should reject further inappropriate changes.

Audit focus:
Verify mutation guards on every write path.
