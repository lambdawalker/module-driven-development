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
| **Release Attestation** | Immutable record per published artifact or production promotion | Binds exact commit SHA, artifact SHA-256 digest, resolved dependency lockfile state, context-isolated verification results, deterministic policy status, and (for deployable releases) human QA authorization (cryptographic signing remains a design follow-up). |

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
| Audit risk tier | Assigned tier (a project-assigned risk class based on capabilities and consequences; see [Audit scope](audit-scope.md)) |

### Illustrative coordinates specification

This is a proposed example, **not a verified specification of the existing coordinates library**.

- Purpose: transform coordinates between image and overlay spaces; camera capture and UI rendering are outside the responsibility.
- Public contract: define coordinate origins, units, rotation, scaling, cropping, and mirroring precisely; zero exported transitive dependencies.
- Durable invariant: a forward transformation followed by its inverse recovers the original point within a stated floating-point tolerance whenever invertible.
- Capability ceiling: pure caller-supplied in-memory values; zero network egress, persistent storage, or device permissions (tier depends on consumer consequences).
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

Verification has complementary responsibilities:

1. **Independent behavioral test design:** a fresh verifier session starts from the approved Module Specification, Change Agreement, and public interfaces, without the implementer's explanations. It derives adversarial, boundary, and property tests before inspecting implementation details.
2. **Implementation and security review:** a reviewer examines source, dependencies, data flows, build scripts, permissions, and deployment configuration. This review needs access to implementation internals to detect hidden logging, unauthorized access, or unsafe failure paths.
3. **Integrated human QA:** a named human exercises the release candidate in its intended context before production promotion.

Context separation reduces one source of correlated mistakes; a second AI session does not prove independence or completeness. Record each review's inputs, scope, findings, and limitations. Implementer-authored tests remain useful supplementary evidence.

Approved acceptance tests and their expected results are protected against silent weakening. CI compares them with an approved baseline outside the candidate's write authority. New tests are welcome; changing approved expectations requires an explicit agreement revision and approval.

### Deterministic merge gates versus human review

Implementation authorization and merge/acceptance authority are distinct. **An implementing AI must never self-authorize automatic merging by classifying its own diff as routine.**

Human-approved policy defines pre-authorized categories, required evidence, escalation conditions, and any narrow exceptions. Deterministic CI checks enforce that policy; they do not establish that all meaningful behavioral changes have been detected.

Default escalation conditions include:

- Changes to the Module Specification, approved agreement criteria, or locked acceptance tests.
- Public API/ABI or wire-schema changes.
- Dependency lockfile, version catalog, build script, setup configuration, or workflow changes.
- Manifest, entitlement, IAM, secret-binding, or network-egress changes.
- Changes to critical-trust modules.
- An explicit human-review flag or unresolved concern raised by a human, verifier, or policy check.

Policy precedence is explicit: an active human-review flag, failed/missing required evidence, or unresolved scope or semantic uncertainty blocks automatic acceptance. A previously human-approved exception may narrow a particular default escalation rule—for example, specified patch dependency updates—but cannot override those blocks. Otherwise, any triggered default rule requires human review. Changes outside an explicitly pre-authorized category also require review.

Behavioral contract changes can preserve API signatures and file paths. Review must consider consequences, including coordinate conventions, data destinations, and authorization semantics. Unexpected behavior or uncertainty discovered during implementation or verification returns to the human even when static gates pass.

Gate definitions, verifier configuration, approved test baselines, and approval identities must come from protected, approved revisions. Candidate changes to those controls cannot validate themselves; approval of a control update does not automatically authorize its use for accepting that same candidate. Record the policy revision, exceptions applied, evidence, and exact candidate accepted.

Only a change within approved scope and a pre-authorized category, with no unresolved escalation and all required evidence passing, may merge automatically. Policy must state whether an existing human approval covers a detected change or whether another decision is required; the presence of an approved specification diff alone should not create an endless approval loop.

### Library artifact publication versus Human QA before production release

**Decision established in discussion:** a named human tests the integrated release candidate and records that it was QA'd and is ready for production. Automatic acceptance and merging do not substitute for this production release decision.

To prevent multi-repository deadlocks while preserving production safety, the methodology distinguishes three publication and promotion stages:

- **Library candidates:** approved candidate-publication policy may allow an unmerged PR or integration branch to publish an immutable artifact into a restricted staging registry after candidate checks pass. This authorizes testing only, not merge, public release, or production use. Candidate workflows receive only staging privileges.
- **Public library versions:** publication follows merge/release policy, required review, and final artifact verification. Consumer evidence must cover the final published artifact and resolved dependencies, or documented equivalence to the tested candidate; a source SHA alone does not prove equivalence. See [Repository automation](repository-automation.md).
- **Deployable application and live backend production release (for example, mobile store builds or `go.attestra.aws` deployments):** promoting an integrated release candidate to production always requires named **Human QA sign-off**.

QA is a responsibility rather than a headcount requirement: a solo developer can serve as the named human tester. Record the exact tested build/digest, scenarios exercised, known limitations, and release decision in the Release Attestation. Bind the decision to the integrated source/artifact identities, actual resolved dependencies, model/assets, configuration, migrations, and target environment. If any relevant part changes afterward, re-evaluate affected checks and QA sign-off; preserve unrelated evidence only with an explicit applicability rationale.

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

### Review capability and remote approval

Approval depends on sufficient evidence, authenticated authority, and the ability to inspect the relevant behavior—not the device used.

A concise Change Agreement or containment decision may be easy to review on a phone. Security-sensitive changes and large diffs need deeper inspection, which may be more convenient on a workstation or remote review environment. Defer approval when the available surface cannot support that inspection; do not make workstation ownership a methodology requirement.

Human QA can occur on the actual target device, including an Android phone. Recording or signing the production decision remotely is valid when the required QA and review evidence are available and bound to the exact release candidate. Signing proves authorization, not testing quality.

### Proposed behavior

1. AI or a deterministic policy gate detects that the next action exceeds existing authorization.
2. AI prepares a concise, reviewable proposal identifying the blocked stage and required evidence and inspection capabilities.
3. The configured channel notifies the authorized human and links to the exact commit/revision.
4. The human approves, rejects, or requests revision.
5. The runner cryptographically or API-verifies the decision, the human identity, and the exact proposal SHA covered.
6. Only the authorized stage resumes. **Silence, session timeout, or notification delivery is never approval.**

While blocked, AI may continue orthogonal work that stays strictly within prior authorization. Any material edit to a pending proposal invalidates prior approval and requires a fresh decision on the new revision.

### Proposed request contents

- Module, repository, exact commit SHA, and required review scope.
- Blocked action and the specific authorization scope requested (implement Change Agreement, merge escalated PR, or execute recovery).
- Summary of scope delta, deterministic gates triggered, security/compatibility impact, and recovery owner.
- Verification evidence summary, open uncertainties, and clear approve / reject / revise actions.

## GitHub as a candidate implementation

A candidate setup uses a dedicated, least-privilege GitHub App or bot account to open pull requests and request review from the human maintainer's account (since GitHub forbids self-approval of one's own pull request).

**Candidate GitHub safeguards:** before enabling autonomous merging or release, verify enforced target-branch rulesets, required status checks, authenticated human approval for escalated changes, stale-approval invalidation, and no automation bypass. A protected policy check must enforce conditional human review; do not assume one static required-review count represents both automatic and human-reviewed acceptance paths. Creating a development branch alone does not require production release controls.

References consulted during discussion:

- [GitHub notification configuration](https://docs.github.com/en/subscriptions-and-notifications/get-started/configuring-notifications)
- [GitHub Mobile](https://docs.github.com/en/get-started/using-github/github-mobile)
- [Approving pull requests with required reviews](https://docs.github.com/en/pull-requests/how-tos/review-pull-requests/approving-a-pull-request-with-required-reviews)

## Required trial and notes (*)

Before relying on remote approval, run and log a trial with one human approver away from the workstation.

| Scenario | What to check |
| --- | --- |
| Request delivery | Notification reaches the phone and opens the exact proposal revision |
| Ergonomic clarity | Requests expose sufficient evidence on the available device or clearly explain what further inspection is needed |
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
