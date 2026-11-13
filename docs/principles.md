# Principles

## Agreed principles

### AI performs most development work

The methodology assumes AI will do most of the development. Modules should provide a scope that AI can understand and work within, and that a reviewer can inspect.

Specific responsibilities for humans, AI implementers, and independent verifiers must keep authorization and verification distinct from implementation.

### Small, well-defined units

Each module has a cohesive responsibility with a boundary that can be explained precisely.

Smallness is evaluated through conceptual complexity, dependencies, interactions, and the context needed to understand and audit the module. A short implementation can still combine unrelated responsibilities. A longer implementation can remain cohesive.

Line count can prompt a review of the boundary; it does not automatically determine when to split a module. No numeric limit has been agreed.

### Independent source control

Each established module is maintained in its own repository. This creates a distinct place to track its changes, access, review history, and releases.

A feature can consist of several modules. Login, for example, could have separate repositories for its backend, web integration, Android integration, and iOS integration. These repositories collectively implement the feature.

The example illustrates possible boundaries; it is not a required repository naming scheme or a rule that every feature must have all four platforms.

### Extract difficult logic for focused verification

A module can be a technical component within a feature. Extraction is justified when it creates a clearer place to reproduce a problem, establish the behavior, and verify the implementation independently.

The coordinates library illustrates this: coordinate logic was causing overlay problems, and separating it allowed focused work on correctness. This motivation comes from the project owner's account; the source study records the library and its consumer integration.

A component does not need a large codebase or multiple consumers to justify an independent repository. After the problem is solved, reintegration is optional. Keep the library boundary when it continues to help understanding, verification, and maintenance and when reintegration offers little value.

Extraction alone does not prove correctness. Verify the isolated behavior and its integration with the consuming library.

### Shared automation makes independent repositories practical

Repository independence should be supported by shared setup and delivery tooling. Automate recurring setup and apply a consistent publication or deployment standard while keeping module-specific configuration explicit.

For Android, `gradle.android-repo-setup` is the intended shared tooling. Adoption is in progress; a target convention is distinct from a completed migration.

### Comprehensive auditability

Auditing covers all aspects of a module: implementation, AI agent threat surfaces, dependencies, contracts, data, permissions, secrets, build and release processes, deployment, version skew, resource budgets, and runtime behavior.

An audit must also examine assumptions and interactions across boundaries. Individually acceptable modules can form an insecure or unstable system.

## Proposed supporting rules

The following rules support the agreed direction and address structural failure modes identified in review.

### Distinguish leaf modules from composition hosts

Not every repository plays the same structural role:

- **Leaf and feature module repositories** implement a single cohesive domain or technical capability (such as `Coordinates`, `Permissions`, or an auth feature library) with explicit ownership and decision responsibilities.
- **Application and composition host repositories** (such as `go.attestra.aws`, mobile application shells, or transitional repositories undergoing extraction) select features, bind shared infrastructure, enforce system-wide budgets, and coordinate deployment.

Composition hosts—and legacy repositories not yet split—may legitimately contain components owned by multiple teams with path-scoped ownership rules, while every module needs clear accountability. Multiple teams may own parts of a cohesive module when their responsibilities and approval rules are explicit; team count alone does not determine a repository boundary.

### Incubate unsettled boundaries before extracting repositories

Splitting a greenfield feature across multiple repositories before its domain concepts and contracts stabilize creates high multi-repo coordination churn.

As an optional project policy, when boundaries are still fluid during day-0 discovery, incubate candidate modules as build-isolated subprojects (enforcing strict package visibility and compiler module boundaries) inside the host or parent repository. Extract a subproject into an independent repository once its contract stabilizes, multiple consumers emerge, access isolation is required, or difficult logic benefits from standalone verification.

### Describe responsibilities, contracts, and resource budgets explicitly

State what a module owns, what it does not own, its inputs and outputs, error behavior, dependencies, security assumptions, and resource ceilings.

A contract must describe behavioral and operational commitments as well as data shapes:

- **Authorization ownership:** an upload endpoint accepting a user identifier must state whether caller ownership is enforced inside the module or guaranteed by an upstream gateway.
- **Cross-boundary version skew:** when deployed backends and installed mobile or web clients update on independent schedules, contracts must define backward-compatibility windows (N-back support), wire-schema evolution rules, and deprecation lifecycles.
- **Resource and resilience budgets:** modules should declare concurrency/thread-pool usage, memory ceilings, binary/artifact size budgets, latency bounds, and retry/backoff behavior so composition hosts can prevent duplicate executors, APK bloat, or cascading retry storms.

### Separate implementation context from verification context

An AI implementer frequently encodes the same flawed assumption into both production code and its accompanying unit tests.

To counter correlated mistakes, derive behavioral tests in a fresh context from the approved specification and contract. Separately review implementation internals, data flows, permissions, and delivery configuration; black-box tests cannot replace that audit. Record limitations of each verification role. Approved acceptance tests and the policies enforcing them must be protected against silent modification by the implementing agent. See [Specifications and approvals](specifications-and-approvals.md#context-isolated-verification).

### Scope access and defend agent boundaries

Give AI development tools, CI jobs, and runtime components only the repository, filesystem paths, network egress, data, and credentials their specific task requires. Repository separation provides security value only when least privilege is mechanically enforced:

- Restrict agent network egress and package installation to approved registries to prevent data exfiltration and hallucinated dependency ("slopsquatting") attacks.
- Treat external issues, pull request comments, third-party source comments, build logs, and shared project-memory text as untrusted inputs to defend against indirect prompt injection.
- Never grant AI coding or review agents access to release signing keys, encrypted setup passphrases, or production credentials.

### Keep governance and runtime boundaries explicit

A repository provides a source-control and access boundary. A package provides a distribution boundary. A process, service, or sandbox may provide a runtime boundary.

Separate repositories do not automatically isolate runtime privileges, require network services, or require separate deployments. Modules can be libraries compiled into one application.

### Preserve a view of the whole system

Maintain enough system context to understand dependencies, trust boundaries, sub-repository ownership in composition hosts, data flows, and compatible versions without requiring every task to inspect every implementation.

A lightweight reference option is declarative, version-controlled metadata aggregated across repositories; see [Project Memory Graph](project-memory-graph.md).

### Use evidence to reconsider boundaries

A candidate boundary should make the responsibility easier to explain, verify, and audit.

Frequent lockstep changes across repositories, circular dependencies, extensive shared state, manual artifact stripping, or the need to inspect many external implementations indicate that a boundary—or its contract encapsulation—needs revision. These are review signals rather than automatic rules to merge or split repositories.
