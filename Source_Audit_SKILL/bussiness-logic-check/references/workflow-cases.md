# Workflow and Lifecycle Source Cases

## Purpose

This file defines workflow-specific source points for business logic source discovery.

Use it when the target workflow involves:
- draft / review / approve / publish
- open / assign / resolve / close
- create / confirm / finalize / archive
- cancel / revert / resubmit
- moderation
- lifecycle transitions with role or state gating

This reference is guidance, not proof. Always verify the real state model, source origin, transition guards, approval rules, and alternate path handling in code.

---

# 1. Workflow Source Discovery Points

Prioritize these source values and events:
- business object ID
- current state or status loaded from storage
- target state, action name, or transition command
- approval, review, confirmation, and completion markers
- actor ID, role, reviewer, approver, assignee, or owner
- terminal-state marker
- batch/admin/import/retry path trigger
- side-effect trigger tied to publish, approve, finalize, archive, or cancel

Source questions:
- Which source selects the workflow object?
- Which source provides current state and target transition?
- Which source proves prerequisites are complete?
- Which source determines actor, role, approver, or assignee?
- Which alternate paths can change the same lifecycle state?

---

# 2. Workflow Source Patterns

## W-S1. Target transition source
Example idea:
- route path or request body supplies `approve`, `publish`, `cancel`, `archive`, `target_state`, or `status`.

Audit relevance:
The requested transition must be checked against current state and workflow rules.

Follow-up:
- verify server-side transition validation and terminal-state protection.

## W-S2. Current state source
Example idea:
- service loads object state from database before mutation.

Audit relevance:
Current state is the authoritative source needed to decide valid transitions.

Follow-up:
- verify the transition actually uses this state before side effects.

## W-S3. Prerequisite source
Example idea:
- approval record, review result, confirmation flag, or completed step marker.

Audit relevance:
Prerequisite sources determine whether later workflow steps may run.

Follow-up:
- verify prerequisite state is server-side and cannot be skipped through alternate entry points.

## W-S4. Actor and role source
Example idea:
- approver ID, reviewer ID, assignee ID, operator role, or owner ID.

Audit relevance:
Workflow transitions often require both role and state consistency.

Follow-up:
- verify role, actor, and state are checked together on every transition path.

## W-S5. Alternate lifecycle source
Example idea:
- admin batch, import, retry, queue, or ops path can update the same status.

Audit relevance:
Alternate sources may bypass the primary workflow validator.

Follow-up:
- compare source origin and rule enforcement across equivalent paths.

---

# 3. What to Inspect in Code

Inspect:
- workflow service methods
- state transition checks
- status update helpers
- role + state combined validation
- terminal-state protection
- retry behavior for final actions
- approval, publish, archive, cancel, and finalize handlers
- batch/admin/ops paths
- queue or callback paths changing lifecycle state
- side effects tied to state changes

---

# 4. Source-Level Danger Signals

High-priority source signals:
- `status` or `target_state` comes directly from request body
- action route updates status without loading current state
- approval and publish sources are handled in separate paths with weak coupling
- terminal state marker is not read before mutation
- admin batch path accepts raw target state
- side-effect trigger is tied to request action rather than validated transition success

---

# 5. Case Templates

## Case WF-S-1: Publish transition source

Source focus:
Identify object ID, publish action source, current state source, and approval prerequisite source.

Follow-up:
Verify publish requires approved/reviewed server-side state.

## Case WF-S-2: Repeated final action source

Source focus:
Identify final action, terminal-state marker, idempotency marker, and side-effect trigger.

Follow-up:
Verify repeated approval/finalization does not repeat business effects.

## Case WF-S-3: Alternate path source

Source focus:
Identify main route and admin/batch/retry/import route that can change the same state.

Follow-up:
Compare whether both paths use the same current-state and transition sources.

## Case WF-S-4: Terminal mutation source

Source focus:
Identify mutation input, current terminal state, and write path.

Follow-up:
Verify archived, finalized, closed, or canceled objects reject inappropriate changes.
