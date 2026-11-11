# Project Memory Graph

**Working name. Status: proposed companion system / concept draft, 2026-11-11.** This document captures an idea for later design and experimentation. No implementation, storage technology, or hosting model has been selected.

## Purpose

Provide durable shared memory of a project's structure, decisions, obligations, versions, and overall progress across interconnected repositories.

A project is larger than a single repository. Its modules evolve independently, publish different versions, and depend on one another. Project Memory Graph would help humans and AI understand that whole without relying exclusively on conversation context or scattered pull request notes. It directly addresses the coordination and version-tracking costs of multiple repositories.

The methodology remains usable without this software. The system may support other development approaches as well.

## Capabilities requested in discussion

- Track module dependencies as a graph.
- Draft and display interactions between modules in diagrams.
- Show the humans and agents responsible for obligations and their status.
- Expose an API and MCP interface for agents and tooling.
- Provide a development dashboard and reports for humans.
- Let AI record project information and retrieve it in later sessions.
- Track overall progress and the versions used across interconnected repositories.

## Proposed records

These are initial modeling suggestions, not a finalized schema.

| Record | Purpose |
| --- | --- |
| Project | Connect repositories and modules to their shared goals |
| Module | Identify responsibility, repository, contract, and maintainers |
| Version or artifact | Identify source revision, published artifact or deployment, and verification evidence |
| Dependency | Record that one module consumes another, including required and actually integrated versions |
| Obligation | Record what one module must deliver for another, why, acceptance criteria, and blockers |
| Assignment | Identify the responsible human or agent and the scope of their responsibility |
| Module agreement | Link to approved scope, constraints, verification cases, and approval record |
| Evidence and decision | Link results and acceptance decisions to the exact versions and obligations they support |
| Progress event | Preserve changes in state and their basis over time |

A dependency is a structural relationship: **A consumes B**. An obligation is a commitment: **B must supply a capability that fulfills A's need**. A dependency can have multiple obligations, and completing one does not complete all work between the modules.

## Example: consumer-led integration

The discussion established two checks: the changed library B verifies its own obligations, and consumer A independently verifies that integrating B fulfills A's motivating need.

A may have an integration branch with tests that intentionally fail while waiting for B's deliverable. That is a legitimate pending state, rather than evidence that those tests should be removed or weakened.

The graph could record:

1. A's motivating requirement, branch, and expected failing checks.
2. The linked obligation assigned to B, its agreed checks, and responsible human or agent.
3. B's resulting revision or artifact and its verification results.
4. A's integration of that exact B version and its separate consumer results.
5. The final acceptance outcome and any remaining blockers.

B can finish its own deliverable while A's integration is pending. B's reported completion must not automatically satisfy A's acceptance criteria. Expected failures explain work in progress; they do not constitute acceptance or authorize merging.

## Version tracking and overall progress

The system should distinguish a requested version, an available candidate, an actually integrated version, and a combination verified together. Availability alone does not establish compatibility.

For a project, it could report the module versions in use, pending upgrades, obligations blocked on unpublished artifacts, and verified integration combinations. Progress should distinguish implementation complete, artifact available, consumer integration pending, and accepted outcome rather than collapsing them into one “done” label.

Exact state names, compatibility representation, and progress calculations remain open. A percentage of closed tasks alone would not establish project readiness.

## Shared memory and interfaces

The dashboard, diagrams, reports, API, and MCP should expose consistent views of the same underlying records.

An AI could retrieve the relevant modules, current agreements, outstanding obligations, owners, expected failures, versions, and evidence when resuming work. Records should retain links to their sources so the agent can inspect the original agreement, pull request, commit, artifact, or result.

GitHub can continue to hold code and reviews while this system connects their meaning across repositories. Which records are authoritative in the graph versus external systems remains a design question; the system should not create competing untraceable versions of the same decision.

Useful queries include:

- What is blocking A, who owns the obligation, and what evidence will unblock it?
- Which B version is A actually using, and which combination was verified?
- What has changed since this agent last worked on the project?
- Which failures are expected while a dependency is pending?
- What needs human approval, and what can proceed within existing authorization?
- Which consumers may be affected by this proposed change?

## Trust and governance proposals

- Distinguish proposed relationships from confirmed relationships, and reported completion from verified acceptance.
- Preserve provenance, revision identity, authorship, timestamps, and history for meaningful updates.
- Make stale or conflicting information visible; missing data must not silently appear as verified progress.
- Scope read and write access to each human's or agent's responsibilities.
- Treat retrieved records as project data, not as authority to override approval requirements.
- Link acceptance to its evidence and approved policy or human decision.
- Do not let an agent's completion claim substitute for independent consumer verification.

## Suggested initial scope

Start with modules, dependencies, obligations, human and agent assignments, versions, status history, and evidence links, exposed through a small API/MCP surface and human-readable dashboard or reports.

This is a suggested starting scope rather than an approved implementation plan. Scheduling, notification delivery, automated code execution, and full release orchestration can be evaluated separately.

## Open design questions

- What is the smallest useful deployment for a solo developer and for a team?
- Which system owns each record, and how are GitHub and other sources reconciled?
- How do agents update records safely when working concurrently?
- How are stale records detected and corrections recorded?
- How are versions, artifacts, deployments, and tested combinations identified?
- How are graph views kept understandable as projects grow?
- What information may be stored, and which permissions protect it?
- How will a trial establish that this memory improves continuity and reduces coordination errors?

Related methodology discussion: [Specifications and approvals](specifications-and-approvals.md).
