# Module-Driven Development

A developing methodology for software built primarily by AI, organized into small, well-defined, independently maintained modules.

**Status: initial discussion draft.** This repository captures the ideas agreed so far and keeps proposals and open decisions explicit. It does not yet define a complete development process.

## Purpose

Make software easier to understand, change, and audit by giving each module a bounded responsibility and its own source-control repository.

Security and comprehensive auditability are the primary motivations. AI is expected to perform most development work. Parallel execution is not the primary objective.

## Agreed foundation

- Split software into small, well-defined units with cohesive responsibilities.
- Maintain those units in independent repositories.
- Start with feature boundaries, such as login or uploads; a feature can be further divided into backend, web, Android, and iOS modules.
- Allow focused technical components to become modules when isolation makes difficult logic easier to solve and verify; keeping the resulting library is valid even with one consumer.
- Automate repository setup and standardize publication and deployment through shared tooling, with rollout tracked per repository.
- Judge boundaries by responsibility, complexity, and independent auditability. Line count alone is a poor measure.
- Audit every aspect of a module, including data access, permissions, dependencies, delivery, and runtime behavior.
- Examine interactions between modules as part of auditing the complete system.

## Approval direction

Humans control meaningful changes, with one human approver as the baseline, including for a developer working alone with AI. One approval of the complete module agreement authorizes implementation. Acceptance can follow a developer-approved automatic policy or explicit human review; a human-review flag blocks automatic merging. Approval must be possible away from the workstation. The remote workflow remains **provisional (*) pending a practical trial and recorded notes**; GitHub is a candidate implementation rather than a platform requirement. Change classification starts with lightweight, context-sensitive guidance that each project refines. Reliable runner pause/resume behavior remains open.

## Read the draft

| Document | Contents |
| --- | --- |
| [Topics needing more detail](docs/topics-needing-detail.md) | Open follow-ups, starting with an artifact-by-artifact recovery discussion |
| [Principles](docs/principles.md) | Module boundaries, platform separation, and what independence means |
| [Benefits and tradeoffs](docs/benefits-and-tradeoffs.md) | Expected advantages, costs, and failure modes |
| [Specifications and approvals](docs/specifications-and-approvals.md) | Module agreement, evidence, automatic acceptance, adaptable change guidance, and provisional remote approval trial |
| [Project Memory Graph](docs/project-memory-graph.md) | Proposed companion dashboard, API/MCP, and shared project memory across repositories, versions, obligations, and progress |
| [Recovery planning guidelines](docs/recovery-planning-guidelines.md) | Artifact-specific recovery prompts, human ownership, and course correction during development |
| [Audit scope](docs/audit-scope.md) | Proposed coverage and evidence for comprehensive audits |
| [Repository automation](docs/repository-automation.md) | Shared setup, artifact publication, and deployment conventions |
| [Auth ecosystem case study](docs/case-studies/auth-ecosystem.md) | Source-backed Android and backend composition example, including coordinates extraction |

## Decisions still open

Track concrete follow-ups in [Topics needing more detail](docs/topics-needing-detail.md), including the requested return to artifact-specific recovery guidance.

- What information and evidence must every module publish?
- How are contracts specified, owned, versioned, and tested across repositories?
- What can AI decide autonomously, and what requires human judgment?
- Who reviews the work, and how is verification kept sufficiently independent of implementation?
- What evidence is required before release, and what changes trigger another audit?
- How are compatible module versions and coordinated changes managed?
- How are module boundaries evaluated as the system evolves?

Module planning, the module agreement, evidence and acceptance paths, and adaptable change guidance are captured in the draft. Detailed release gates and a complete development lifecycle remain to be defined.
