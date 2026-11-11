# Project Memory Graph

**Working name. Status: proposed companion system / concept draft, 2026-11-11.** This document captures an architecture concept for staged experimentation. No custom database or hosted service has been deployed.

## Purpose

Provide durable shared memory of a project's structure, specifications, change agreements, obligations, versions, and overall progress across interconnected repositories.

A project is larger than a single repository. Its modules evolve independently, publish candidate and release artifacts, and depend on one another. Project Memory Graph helps humans and AI understand the assembled system without relying on ephemeral chat history or scattered pull request comments, directly addressing the coordination cost of multi-repository development.

The methodology remains usable without a dedicated graph service—starting from declarative files in Git—and the graph model can support other development workflows as well.

## Capabilities requested in discussion

- Track module dependencies and exported contracts as a graph.
- Draft and display architectural and interaction diagrams across modules.
- Show the humans and agents responsible for obligations and their current status.
- Expose a schema-validated API and MCP interface for agents and tooling.
- Provide a development dashboard and progress reports for humans.
- Let AI record structured project updates and retrieve verified state in later sessions.
- Track overall progress, staging candidates, and exact versions integrated across repositories.
- Support planning-time impact analysis and participation from affected teams, including multi-team composition hosts.

## Proposed records

Aligned with the three-artifact governance model in [Specifications and approvals](specifications-and-approvals.md), initial modeling separates durable module identity from task agreements and immutable release proofs:

| Record | Purpose |
| --- | --- |
| Project | Connect leaf module repositories and application composition hosts to shared goals |
| Module | Identify repository (or sub-repository path in a host), role (leaf module vs. composition host), owners, and risk tier |
| Module specification | Link to the versioned durable specification (`MODULE.md`), public contracts, capability ceiling, and resource budgets |
| Version or artifact | Identify source SHA, staging candidate or public artifact digest, and N-back compatibility metadata |
| Dependency | Record that consumer A uses provider B, distinguishing declared range, staging candidate, and integrated release version |
| Obligation | Record what B must deliver for A, motivating need, consumer acceptance checks, and active blockers |
| Change agreement | Link a task/PR scope delta, impact assessment, recovery plan, required reviewers, and approval status |
| Assignment | Identify the responsible human or agent and the bounded scope of their authorization |
| Release attestation | Link immutable verification results, audit records, and Human QA sign-offs to exact artifact digests |
| Progress or signal event | Preserve timestamped state transitions, external monitor alerts, and their provenance |

A dependency is a structural relationship (**A consumes B**). An obligation is a commitment (**B must supply a capability that fulfills A's need**). Completing one obligation does not imply all integration work is complete.

## Example: consumer-led integration

Two independent checks govern cross-module work: provider library B verifies its own contract and invariants, and consumer A independently verifies that integrating B satisfies A's motivating need.

A may maintain an integration branch with tests that intentionally fail while waiting for B's deliverable—or point temporarily at a commit-SHA staging build from B's CI inner loop. That pending state is normal work in progress, never a reason to delete or weaken A's tests.

The graph records:

1. A's motivating requirement, integration branch, and expected failing consumer checks.
2. The linked obligation assigned to B, agreed Change Agreement checks, and responsible owner.
3. B's candidate staging artifact (for inner-loop verification) followed by B's confirmed public release version and Release Attestation.
4. A's integration of that exact public B version and A's independent consumer test results.
5. Final acceptance outcome and any remaining production QA gates.

B's completion claim never auto-satisfies A's consumer acceptance criteria.

## Impact analysis during module planning

The graph informs planning before implementation or publication, surfacing direct and transitive consumers, currently pinned versions, N-back client skew constraints, and the application compositions where modules meet.

### Reconciling repository independence with multi-team ownership

As defined in [Principles](principles.md#distinguish-leaf-modules-from-composition-hosts), **leaf and feature module repositories** maintain single-team ownership over one cohesive responsibility. However, **application composition hosts** (such as `go.attestra.aws` or unified mobile shell apps) and legacy repositories undergoing gradual extraction often bring together components owned by different teams across locations or time zones.

Ownership in the graph must therefore support optional **sub-repository path/component granularity** (mirroring `CODEOWNERS` or subproject boundaries) inside composition hosts and transitional repos, rather than assuming repository-level membership alone identifies every stakeholder.

### Example: shared dependency, two teams

Team A requests a change to a shared library. Team B owns another component (either in its own repository or inside a shared application composition host) that consumes that library, and a planned application release integrates both.

During module planning, AI should:

1. Query both consumer relationships, their component owners, pinned versions, and target application composition.
2. Explain potential contract, binary, wire-skew, or resource-budget impacts and supporting evidence.
3. Identify Team B for asynchronous participation in the Change Agreement review via an authorized notification channel.
4. Incorporate both teams' consumer checks into the Change Agreement or linked obligations.
5. Record explicit team-representative review requirements before breaking or cross-cutting changes publish or integrate.

Publishing a new library version and upgrading a consuming application remain distinct events; both must be analyzed.

### Reviews reflect affected ownership

One human approver remains the baseline for a solo developer or single-owner leaf module. When impact analysis shows two teams' components are affected, policy can require one authorized reviewer per affected team—matching actual ownership rather than counting arbitrary approvals. Missing responses are never treated as consent.

### Incomplete knowledge is visible

Records must carry provenance and freshness timestamps. Stale or unindexed consumer relationships mean impact analysis has explicit blind spots—not that zero teams are affected—and those gaps must be surfaced in the Change Agreement.

## Version tracking and overall progress

Progress views must distinguish: `agreement approved`, `implementation complete`, `staging candidate verified`, `public artifact released`, `consumer integrated`, and `production QA signed off`, rather than collapsing multi-repo readiness into a single "done" percentage.

## Trust, governance, and prompt-injection defenses

Because AI agents read Project Memory Graph records via MCP during planning and implementation, the graph is a high-value target for **indirect prompt injection** and state poisoning:

- **Treat free-text fields as untrusted data:** issue summaries, external user reports, commit notes, and free-form descriptions retrieved via MCP must be delineated as untrusted data and never interpreted as system instructions or authorization overrides.
- **Use strict typed schemas for control state:** status transitions, approval flags, risk tiers, artifact digests, and reviewer identities must use schema-validated enums, cryptographic hashes, and authenticated principal IDs rather than free-text prose.
- **Scope write permissions per repository and role:** an agent working on Module B may post candidate build links for Module B, but cannot mark Consumer A's obligation as accepted or clear a human-review requirement.
- **Single source of authority:** Git repositories and enforced CI/review records remain authoritative for code, specifications, and approvals; the graph indexes and links to those immutable URLs/SHAs rather than creating a competing approval ledger.

## Phased bootstrap and operational monitoring boundary

### Phase 1: Git-native declarative manifests (recommended solo/small-team start)

Avoid operating a separate database and stateful web service before repository count demands it:

1. Each repository checks in a small, schema-validated manifest (for example, `MODULE.yaml` or frontmatter in `MODULE.md`) declaring module ID, role (leaf vs. host), owner, risk tier, exported contracts, and consumed module coordinates.
2. A lightweight CI job or local CLI aggregates manifests and GitHub PR/release metadata across repositories into a deterministic JSON graph artifact exposed via a local MCP server and static HTML/Markdown summary.
3. Git remains the sole source of truth, eliminating split-brain reconciliation and concurrent database write races.

### Phase 2: Dedicated coordination service

Introduce persistent storage, live webhook ingestion, cross-repo obligation queues, and interactive dashboards only when multi-team concurrency or cross-repo query volume outgrows static CI aggregation.

### Operational monitoring boundary

Project Memory Graph is an architectural and coordination memory system, **not** a primary runtime monitoring engine. Periodic operational checks (such as verifying S3 bucket public-access blocks, database health, or error-rate spikes) belong in dedicated infrastructure and cloud tools (AWS Config, CloudWatch, Prometheus). Those external monitors should emit structured drift or alert events *into* the graph so the graph can map affected modules, consuming teams, and responsible recovery owners.

## Open design questions

- Exact YAML/JSON schema for Phase 1 in-repo manifests and obligation links.
- Reconciliation rules if Phase 2 introduces mutable operational annotations alongside Git-owned specifications.
- Diagram generation rules that stay readable as module count grows.
- Trial metrics proving reduced cross-repo coordination errors compared to plain repository docs.

Related methodology discussion: [Specifications and approvals](specifications-and-approvals.md) and [Principles](principles.md).
