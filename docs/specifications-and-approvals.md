# Module specifications and human approval

**Status: discussion draft — 2026-11-11.** This document captures agreed governance decisions and proposed structural safeguards. The remote approval workflow remains **provisional (*) and must be tested before being treated as a validated practice**.

## Decisions established in discussion

- Humans retain control over meaningful changes. AI must request a decision when required approval blocks further work.
- One explicit human approval of a planned agreement authorizes implementation against that approved scope.
- Acceptance of completed work and promotion to production are distinct from implementation authorization; named human QA is required before promoting a deployable release candidate to production.
- Approval must be possible while the developer is away from their workstation, for example from a phone with connectivity, for decisions suited to mobile review.
- Start with **one human approver**, supporting teams consisting of one developer working with AI, scaling review representation when multiple teams are affected.
- The methodology must accommodate the developer's infrastructure; GitHub is a candidate implementation, not a mandatory platform.
- Keep the remote approval section in draft until a practical trial is completed. Record outcomes and friction, then revisit the methodology.

These decisions describe the intended process. No notification service, separate automation identity, repository ruleset, or runner integration was configured or tested during this discussion.

## Three-artifact governance split

To avoid conflating enduring module boundaries with per-task plans or point-in-time release proofs, module governance separates three distinct, linked records:

| Record | Lifecycle and location | Purpose and contents |
| --- | --- | --- |
| **Module Specification** | Durable, living file versioned in the repository (for example, `MODULE.md` or structured manifest) | Defines enduring responsibility, explicitly excluded scope, public contracts, durable invariants, forbidden capabilities and access ceiling, resource budgets, and audit risk tier. |
| **Change Agreement** | Per-task or per-PR record produced during **module planning** | Defines the bounded scope delta, motivating obligation, new or updated acceptance cases, consumer impact analysis, and change-specific recovery and containment plan. |
| **Release Attestation** | Immutable record per published artifact or production promotion | Binds exact commit SHA, artifact SHA-256 digest, resolved dependency lockfile state, context-isolated verification results, deterministic policy status, and (for deployable releases) signed human QA authorization. |

When creating a brand-new module, initial module planning establishes both the initial **Module Specification** and first **Change Agreement** under a single human approval. For existing modules, routine features or fixes require only a **Change Agreement** that stays within the durable **Module Specification**; any task that alters the durable specification's boundaries, public contracts, or access ceiling explicitly flags that specification diff for human approval.

Precise file schemas and filenames remain open.

### Proposed Module Specification contents

| Area | Proposed contents |
| --- | --- |
| Purpose and boundaries | Cohesive responsibility, intended consumers, and explicitly excluded responsibilities |
| Public contract | Exported APIs/schemas, units and conventions, error semantics, N-back compatibility commitments, and caller obligations |
| Durable invariants | Properties that must always hold, with measurable tolerances where relevant |
| Capability and access ceiling | Permitted data access, permissions, credentials, storage, network egress, and dependency constraints |
| Resource budgets | Concurrency/thread-pool model, memory and binary-size ceilings, latency targets, and retry/backoff bounds |
| Audit risk tier | Assigned tier (Tier 1 critical trust, Tier 2 stateful/networked feature, or Tier 3 pure-compute leaf; see [Audit scope](audit-scope.md)) |

### Illustrative coordinates specification

This is a proposed example, **not a verified specification of the existing coordinates library**.

- Purpose: transform coordinates between image and overlay spaces; camera capture and UI rendering are outside the responsibility.
- Public contract: define coordinate origins, units, rotation, scaling, cropping, and mirroring precisely; zero exported transitive dependencies.
- Durable invariant: a forward transformation followed by its inverse recovers the original point within a stated floating-point tolerance whenever invertible.
- Capability ceiling: pure caller-supplied in-memory values; zero network egress, persistent storage, or device permissions (`Tier 3` pure-compute leaf).
- Resource budget: zero background threads, zero heap retention across calls, minimal JAR footprint.
- Example planning question for a Change Agreement: whether points outside the image bounds clamp, error, or pass through.

## Module planning and the Change Agreement

The planning conversation is called **module planning**. Its approved output for a unit of work is the **Change Agreement** (historically referred to in early discussion notes as the module agreement).

The human and AI craft the draft together, covering what will change, whether the durable Module Specification is touched, concrete acceptance criteria, the automated or manual evidence that will demonstrate success, and recovery/containment expectations. A human prompt can start as a single paragraph; AI expands candidate edge cases, consumer impacts, and verification plans for review.

One explicit human approval covers the complete draft and authorizes implementation against that revision. Separate approvals for each individual test case are not required. Routine implementation details remain within approved scope, while material discoveries or scope changes pause the affected work for course correction; open assumptions must remain explicitly flagged rather than silently treated as approved.

### Recovery and course correction

During planning, AI must raise **“What do we do if this fails?”** if humans do not, identifying a named human who accepts recovery responsibility and distinguishing any pre-authorized break-glass containment (such as a kill switch) from human-authorized state recovery. Use [Recovery planning guidelines](recovery-planning-guidelines.md) for artifact-specific prompts.

If implementation reveals an unanticipated requirement—such as a new database table, migration, or cross-team contract change—developer and AI hold a course-correction conversation, reassess impact and recovery, involve newly affected teams through an authorized channel, and approve the revised Change Agreement before resuming the blocked work.

## Evidence, context-isolated verification, and acceptance

Verification can combine deterministic static checks, automated test suites, context-isolated adversarial verification, and manual inspection.

### Context-isolated verification

Because an implementing AI agent tends to write both implementation and unit tests around the same blind spots, non-trivial modules should separate **implementation** from **independent verification**:

1. Acceptance criteria and locked contract tests agreed during planning are protected by CI path/hash checks so the implementing agent cannot weaken, delete, or rewrite them to pass.
2. An independent verifier step (or separate clean-context AI session) receives **only** the durable Module Specification, the approved Change Agreement, and the public contract/interface—never the implementer's chain-of-thought, commit commentary, or private implementation internals—and authors adversarial edge cases, negative boundary checks, and invariant/property tests.
3. Implementer-authored unit tests remain welcome as supplementary coverage alongside the locked and verifier-authored suites.

### Deterministic merge gates versus human review

Implementation authorization and merge/acceptance authority are distinct. **An implementing AI must never self-authorize automatic merging by classifying its own diff as routine.**

While AI may include a contextual risk summary in the pull request description to assist reviewers, eligibility for automatic acceptance is enforced strictly by **deterministic CI policy gates** configured under a human-approved repository policy. Any of the following diff conditions automatically sets the human-review block and disables automatic merging:

- Edits to the durable `Module Specification`, Change Agreement criteria, or locked acceptance test files.
- Public API/ABI surface changes (detected via API dump tools such as `binary-compatibility-validator`, OpenAPI/Protobuf schema diffs, or exported symbol checks).
- Modifications to dependency lockfiles, version catalogs, build scripts, repository setup configs, or CI workflow definitions.
- Changes touching platform manifests or entitlements (such as `AndroidManifest.xml`), IAM/Pulumi infrastructure policies, secret bindings, or network egress rules.
- Any touch within a `Tier 1` critical-trust module unless explicitly scoped by a narrower, human-signed rule.
- Presence of an explicit human-review label/flag set by a human, verifier agent, or policy check.

Only when a change falls strictly within a pre-authorized category, triggers zero deterministic escalation gates, and passes all required build, contract, and context-isolated verification checks may CI merge automatically into the target branch.

### Library artifact publication versus Human QA before production release

**Decision established in discussion:** a named human tests the integrated release candidate and records that it was QA'd and is ready for production. Automatic acceptance and merging do not substitute for this production release decision.

To prevent multi-repository deadlocks while preserving production safety, the methodology distinguishes two deliverable classes:

- **Library module publication (for example, `Coordinates` or `Card Detector`):** once code merges under the applicable review and deterministic verification gates, automation may publish immutable commit-SHA staging builds—or policy-governed library versions—so waiting downstream consumer repositories can integrate and run their own obligation checks (see [Repository automation](repository-automation.md)). Changes altering library security boundaries or public contracts still require human review prior to public release.
- **Deployable application and live backend production release (for example, mobile store builds or `go.attestra.aws` deployments):** promoting an integrated release candidate to production always requires named **Human QA sign-off**.

QA is a responsibility rather than a headcount requirement: a solo developer can serve as the named human tester. Record the exact tested build/digest, scenarios exercised, known limitations, and release decision in the Release Attestation. If the candidate changes afterward, affected checks and QA sign-off must be re-evaluated.

A later design discussion will bind internal production authorization to the Release Attestation digest using a phone or hardware security key; see [Signed production authorization](topics-needing-detail.md#signed-production-authorization).

## Context-sensitive change guidance

**Agreed direction: lightweight starting guidance for every project, refined as that project develops. These are guidelines for human policy authors and reviewer recommendations, not self-granted AI authority.**

AI can propose how a novel change category should be classified and explain the reasoning from project context, while humans decide policy updates and translate repeatable rules into deterministic CI gates.

Judge significance by consequences rather than line count:

- Who gains access or privileges?
- What data moves, and does it reach a new destination or sensitivity class?
- What behavior, wire contract, or invariant do users and consumers rely on?
- How broad is the blast radius, and can affected clients or state be recovered?

| Example | Context and policy guidance |
| --- | --- |
| New mobile permission | Manifest diff deterministically blocks auto-merge; requires human approval |
| New public comic-store content page | May qualify for automatic acceptance when confined to content paths within approved scope and checks |
| New site URL or backend route | Route/schema diff triggers review if it exposes client documents, auth, or external effects |
| Patch-level internal dependency bump | May follow an approved automatic policy only if lockfile/advisory/compatibility gates explicitly allow patch bumps from trusted registries |
| Dependency altering auth, licensing, or network calls | Escalates to human review via license/capability/advisory gates |
| Coordinates internal refactor | Qualifies for automatic acceptance only when public API dump, locked invariant tests, and verifier checks are unchanged and green |
| Coordinate-origin or mirroring change | Alters behavioral semantics; requires human review and major/contract version handling even if a two-line diff |

## Remote approval workflow (*)

**Draft pending practical validation.**

### Approval surface tiers (mobile versus workstation)

Being away from the workstation should not stall routine agent progress, yet tiny screens invite rubber-stamping on complex diffs. Split remote decisions by review ergonomics:

- **Mobile-suitable approvals:** approving a concise, well-bounded Change Agreement within existing module boundaries; answering an explicit open product decision; authorizing a pre-planned containment/rollout pause; or rejecting an off-track proposal.
- **Workstation-required approvals:** changes modifying a durable Module Specification's security ceiling, IAM/permission policies, cryptographic or authentication logic, shared CI/publishing machinery, large multi-file behavioral diffs, and hands-on Human QA production sign-offs.

### Proposed behavior

1. AI or a deterministic policy gate detects that the next action exceeds existing authorization.
2. AI prepares a concise, reviewable proposal identifying the blocked stage and required review tier (mobile-suitable vs. workstation).
3. The configured channel notifies the authorized human and links to the exact commit/revision.
4. The human approves, rejects, or requests revision.
5. The runner cryptographically or API-verifies the decision, the human identity, and the exact proposal SHA covered.
6. Only the authorized stage resumes. **Silence, session timeout, or notification delivery is never approval.**

While blocked, AI may continue orthogonal work that stays strictly within prior authorization. Any material edit to a pending proposal invalidates prior approval and requires a fresh decision on the new revision.

### Proposed request contents

- Module, repository, exact commit SHA, and approval tier (mobile vs. workstation).
- Blocked action and the specific authorization scope requested (implement Change Agreement, merge escalated PR, or execute recovery).
- Summary of scope delta, deterministic gates triggered, security/compatibility impact, and recovery owner.
- Verification evidence summary, open uncertainties, and clear approve / reject / revise actions.

## GitHub as a candidate implementation

A candidate setup uses a dedicated, least-privilege GitHub App or bot account to open pull requests and request review from the human maintainer's account (since GitHub forbids self-approval of one's own pull request).

**Mandatory repository prerequisite:** before enabling autonomous agent branches or auto-merge workflows, target branches (`main` and release branches—including in the methodology repository itself) must enable enforced branch rulesets requiring pull requests, passing status checks, non-author human approval when escalated, stale-review dismissal on new pushes, and zero bypass permissions for the automation identity.

References consulted during discussion:

- [GitHub notification configuration](https://docs.github.com/en/subscriptions-and-notifications/get-started/configuring-notifications)
- [GitHub Mobile](https://docs.github.com/en/get-started/using-github/github-mobile)
- [Approving pull requests with required reviews](https://docs.github.com/en/pull-requests/how-tos/review-pull-requests/approving-a-pull-request-with-required-reviews)

## Required trial and notes (*)

Before relying on remote approval, run and log a trial with one human approver away from the workstation.

| Scenario | What to check |
| --- | --- |
| Request delivery | Notification reaches the phone and opens the exact proposal revision |
| Ergonomic clarity | Mobile-tier requests are understandable on a phone; workstation-tier requests clearly defer to desktop review |
| Approval binding | Runner verifies the authorized human identity and exact commit SHA before resuming |
| Rejection | Blocked action stays halted and records rejection cleanly |
| Revision invalidation | Pushing a modified commit automatically dismisses or invalidates an earlier approval |
| No response | Blocked action stays paused indefinitely; orthogonal authorized tasks can proceed |
| Resume or reconnect | Runner restart preserves pending state and enforces idempotency so authorized actions never run twice |

Record date, repository, bot/human identities, ruleset configuration, commit SHAs, runner behavior, and friction notes before removing the provisional marker. No trial results exist yet.

## Remaining discussion

- Standard file schemas for `Module Specification`, `Change Agreement`, and `Release Attestation`.
- Mechanical enforcement of context isolation between implementer and verifier agents.
- Runner pause, webhook/polling verification, and idempotent resume implementation.
- Detailed re-audit triggers and Maintenance lifecycle; see [Audit scope](audit-scope.md) and [Topics needing more detail](topics-needing-detail.md).
