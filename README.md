# Modular Assurance

A methodology for accountable AI software development.

Modular Assurance organizes software built primarily by AI into small, well-defined, independently maintained modules, supported by explicit agreements, verification evidence, and human accountability.

**Status: initial discussion draft.** This repository captures the ideas agreed so far and keeps proposals and open decisions explicit. It does not yet define a complete development process.

## Purpose

Make software easier to understand, change, and audit by giving each module a bounded responsibility and its own source-control repository.

Security and comprehensive auditability are the primary motivations. AI is expected to perform most development work. Parallel execution is not the primary objective.

## Agreed foundation

- Split software into small, well-defined units with cohesive responsibilities.
- Maintain leaf and feature modules in independent repositories, while distinguishing application composition hosts that assemble them.
- Start with feature boundaries, such as login or uploads; a feature can be further divided into backend, web, Android, and iOS modules, with optional incubation policies described separately.
- Allow focused technical components to become modules when isolation makes difficult logic easier to solve and verify; keeping the resulting library is valid even with one consumer.
- Automate repository setup and standardize publication and deployment through shared tooling, with rollout tracked per repository.
- Judge boundaries by responsibility, complexity, and independent auditability. Line count alone is a poor measure.
- Audit every aspect of a module—proportional to its risk tier—including AI agent threat surfaces, data access, permissions, dependencies, delivery, version skew, resource budgets, and runtime behavior.
- Examine interactions between modules as part of auditing the complete system.

## Core, project policy, and reference tooling

The core requires explicit module boundaries, comprehensive audit scope, human-authorized work, traceable evidence, and named human QA before production. Projects select review depth, automation categories, compatibility windows, and recovery permissions through explicit human-approved policy.

Risk-tier tables, subproject incubation, ownership arrangements, Git-backed graph aggregation, BOMs, and particular CI or signing tools are proposals or reference implementations. They are not universal prerequisites. A written design is not evidence that its controls have been implemented or validated.

## Governance and approval direction

Humans control meaningful changes, with one human approver as the baseline, including for a developer working alone with AI. Governance separates three distinct records: a living **Module Specification** (durable boundaries, contracts, invariants, and access ceilings), a task-scoped **Change Agreement** (planned delta, acceptance cases, and recovery plan), and an immutable **Release Attestation** (artifact digest, verification evidence, and release sign-off).

One human approval of a Change Agreement authorizes implementation of that delta. Eligibility for automatic acceptance versus mandatory human review is enforced by **deterministic CI policy gates** (such as public API/ABI diffs, manifest or permission changes, lockfile updates, and locked test-suite guards) rather than AI self-classification, paired with independent behavioral test design and implementation/security review. Gates enforce approved rules; semantic uncertainty still requires escalation.

Library modules may publish verified candidate or versioned artifacts to unblock downstream consumer integration, whereas **deployable applications and live backend services require named Human QA sign-off** on the integrated release candidate before production promotion. Approval and QA may occur remotely, including on a phone, when the reviewer can inspect sufficient evidence and relevant behavior. Complex reviews require adequate capabilities rather than a prescribed device. The remote approval workflow remains **provisional (*) pending a practical trial and recorded notes**; GitHub is a candidate implementation rather than a platform requirement.

## Read the draft

| Document | Contents |
| --- | --- |
| [Topics needing more detail](docs/topics-needing-detail.md) | Open follow-ups, including artifact recovery, operational signal routing, and standard-based signed production authorization |
| [Principles](docs/principles.md) | Module boundaries, leaf vs. host repositories, day-0 incubation, context-isolated verification, skew, and resource budgets |
| [Benefits and tradeoffs](docs/benefits-and-tradeoffs.md) | Expected advantages, costs, AI-specific failure modes, and structural countermeasures |
| [Specifications and approvals](docs/specifications-and-approvals.md) | Three-artifact governance split, deterministic merge gates, context-isolated verification, library vs. app QA, and capability-based remote approval |
| [Project Memory Graph](docs/project-memory-graph.md) | Proposed companion graph, MCP prompt-injection boundaries, Git-native Phase 1 bootstrap, and impact analysis |
| [Recovery planning guidelines](docs/recovery-planning-guidelines.md) | Pre-authorized break-glass containment vs. human-authorized recovery, client skew, and mid-sprint course correction |
| [Audit scope](docs/audit-scope.md) | Comprehensive audit domains, AI agent threat model, proposed risk-based depth, and delta re-audit triggers |
| [Repository automation](docs/repository-automation.md) | Shared setup, cross-repo pre-release inner loop, transitive dependency governance, and signing-key isolation |
| [Auth ecosystem case study](docs/case-studies/auth-ecosystem.md) | Source-backed Android and backend composition example, motivating transitive governance and staging rules |

## Decisions still open

Track concrete follow-ups in [Topics needing more detail](docs/topics-needing-detail.md), including the requested return to artifact-specific recovery guidance.

With the three-artifact governance split, deterministic merge-gate principle, context-isolated verification, and production Human QA gate captured in draft, the primary open engineering questions are:

- What exact file schemas and repository paths standardize the Module Specification, Change Agreement, and Release Attestation?
- How should runners implement reliable pause, cryptographic decision verification, and idempotent resume for remote approvals?
- How is context isolation enforced mechanically between implementer and verifier agent sessions?
- Which project-specific risk thresholds and delta conditions automatically trigger a renewed human security audit?
- How will shared automation manage ephemeral cross-repository staging feeds, ecosystem BOM alignment, and N-back wire-compatibility checks?
- Which empirical signals govern graduating an incubated subproject into its own repository or consolidating over-fragmented modules?
