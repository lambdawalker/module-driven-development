# Module specifications and human approval

**Status: discussion draft — 2026-11-11.** This document captures the conversation, distinguishing decisions from proposals. The remote approval workflow is **provisional (*) and must be tested before being treated as a validated practice**.

## Decisions established in discussion

- Humans retain control over meaningful changes. AI must request a decision when required approval blocks further work.
- Approval must be possible while the developer is away from their workstation, for example from a phone with connectivity.
- Start with **one human approver**, supporting teams consisting of one developer working with AI.
- The methodology must accommodate the developer's infrastructure; GitHub is a candidate implementation, not a mandatory platform.
- Keep the remote approval section in draft until a practical trial is completed. Record outcomes and friction, then revisit the methodology.
- Flag additional provisional points as discussion proceeds.

These decisions describe the intended process. No notification service, separate automation identity, repository protection, or runner integration was configured or tested during this discussion.

## Proposed module specification

Keep a short, versioned specification inside each module repository. Its purpose is to bound the AI assignment and provide a basis for review. The precise filename, schema, and approval granularity remain open.

| Area | Proposed contents |
| --- | --- |
| Purpose and boundaries | Responsibility, intended consumers, and explicitly excluded responsibilities |
| Contract | Inputs, outputs, units and conventions, error behavior, compatibility expectations, and caller obligations |
| Invariants | Properties that must remain true, with measurable tolerances where relevant |
| Access and dependencies | Data access, permissions, credentials, storage, network access, and required dependencies |
| Acceptance evidence | Known examples with independently justified expected results, relevant failure cases, and checks of the contract |
| Open decisions | Unresolved assumptions and decisions that require human input before dependent work proceeds |

### Illustrative coordinates specification

This is a proposed example, **not a verified specification of the existing coordinates library**.

- Purpose: transform coordinates between image and overlay spaces; camera capture and drawing are outside the proposed responsibility.
- Contract: define coordinate origins, units, rotation, scaling, cropping, and mirroring precisely.
- Example invariant: a forward transformation followed by its inverse recovers the original point within a stated tolerance, where the transformation is invertible.
- Proposed access profile: operate on caller-supplied values, without network access, persistent storage, or camera permission.
- Proposed evidence: independently established reference examples, rotations and image sizes, and invalid-input behavior.
- Example open decision: whether points outside the image are accepted.

Approval of an entire specification versus approval of selected boundaries and constraints has **not** been settled.

## Remote approval workflow (*)

**Draft pending practical validation.**

### Proposed behavior

1. AI detects that its next action exceeds existing authorization.
2. AI prepares a concrete, reviewable proposal and records which action is blocked.
3. The configured channel notifies the authorized human and provides access to the proposal.
4. The human approves, rejects, or requests a revision.
5. The runner verifies the decision, its author, and the proposal revision it covers.
6. Only the authorized action resumes. The request, decision, and resulting change remain traceable.

While waiting, AI may continue independent work within its current authorization. The blocked action remains paused. **Silence, an expired session, or notification delivery is not approval.**

An approval should identify the exact proposal and authorized stage. A material revision requires another decision. Approval to implement a specification and approval to publish or deploy its implementation are distinct scopes; the exact gate policy remains to be defined.

### Proposed request contents

- Module, repository, and proposal revision or commit.
- The action requiring approval and the stage it authorizes.
- What changes, why the change is needed, and what is blocked.
- Security, data access, compatibility, and operational implications.
- Available evidence, remaining uncertainty, and the AI's recommendation.
- A way to approve, reject, or request revision.

The human should be able to understand the decision from a phone and inspect the underlying proposal as needed.

### Candidate meaningful changes

The following are initial proposals, not a finalized classification:

- Changes to module responsibility or boundaries.
- Changes to public contracts or compatibility commitments.
- Expanded permissions, credentials, or data access.
- Destructive operations.

Routine implementation details may remain within prior authorization. The precise boundary between routine work and approval-required changes is an open decision.

## GitHub as a candidate implementation

A possible implementation uses a dedicated GitHub App or bot identity to prepare a pull request and request review from the developer's personal account. A specification-only pull request could request authorization to begin implementation.

GitHub Mobile can notify users of review requests when configured. Repository rules must enforce the required human review, and the automation identity must not be able to bypass that requirement. The runner separately needs a supported way to detect the decision and resume work.

GitHub does not allow pull request authors to approve their own pull requests, so using the developer's identity for both proposal creation and human review would not provide this review separation.

This is an implementation candidate. Identity setup, available repository rules, least-privilege permissions, mobile notification behavior, and runner integration must be validated in the target environment.

References consulted during discussion:

- [GitHub notification configuration](https://docs.github.com/en/subscriptions-and-notifications/get-started/configuring-notifications)
- [GitHub Mobile](https://docs.github.com/en/get-started/using-github/github-mobile)
- [Approving pull requests with required reviews](https://docs.github.com/en/pull-requests/how-tos/review-pull-requests/approving-a-pull-request-with-required-reviews)

## Required trial and notes (*)

Before relying on this workflow, run a trial with one human approver away from the workstation.

| Scenario | What to check |
| --- | --- |
| Request delivery | Notification reaches the phone and opens the correct proposal |
| Decision clarity | The developer can understand the scope, implications, and evidence |
| Approval | The runner verifies the authorized human and exact revision, then resumes the permitted action |
| Rejection | The blocked action does not run |
| Revision | Approval for an earlier proposal does not authorize a materially revised proposal |
| No response | The action remains blocked; independent authorized work can continue |
| Resume or reconnect | A runner interruption preserves the pending request and does not duplicate the authorized action |

The reconnect scenario is an additional proposed robustness check, not a completed test.

For each trial, record the date, repository, identities, notification channel, request revision, decision, runner behavior, what worked, what failed, and friction or suggested changes. No trial results exist yet.

After the trial, review the notes and explicitly decide which parts to retain or revise before removing the provisional marker.

## Remaining discussion

- Exact criteria for meaningful changes and specification approval granularity.
- Reliable pause, decision detection, and resume behavior.
- Identity setup and enforcement for each supported infrastructure.
- Evidence required before release — the next topic, not yet agreed.
- Later topics: cross-module integration rules, maintenance, re-audit triggers, and recovery.

Earlier lifecycle suggestions (specification, implementation, testing including adversarial cases, independent audit, and release) remain proposals rather than an adopted complete lifecycle.
