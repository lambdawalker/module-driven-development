# Principles

## Agreed principles

### AI performs most development work

The methodology assumes AI will do most of the development. Modules should provide a scope that AI can understand and work within, and that a reviewer can inspect.

Specific responsibilities for humans, AI implementers, and reviewers remain to be defined.

### Small, well-defined units

Each module has a cohesive responsibility with a boundary that can be explained precisely.

Smallness is evaluated through conceptual complexity, dependencies, interactions, and the context needed to understand and audit the module. A short implementation can still combine unrelated responsibilities. A longer implementation can remain cohesive.

Line count can prompt a review of the boundary; it does not automatically determine when to split a module. No numeric limit has been agreed.

### Independent source control

Each module is maintained in its own repository. This creates a distinct place to track its changes, access, review history, and eventual releases.

A feature can consist of several modules. Login, for example, could have separate repositories for its backend, web integration, Android integration, and iOS integration. These repositories collectively implement the feature.

The example illustrates possible boundaries; it is not a required repository naming scheme or a rule that every feature must have all four platforms.

### Comprehensive auditability

Auditing covers all aspects of a module: implementation, dependencies, contracts, data, permissions, secrets, build and release processes, deployment, and runtime behavior.

An audit must also examine assumptions and interactions across boundaries. Individually acceptable modules can form an insecure system.

## Proposed supporting rules

The following rules support the agreed direction but remain proposals.

### Describe responsibilities and contracts explicitly

State what a module owns, what it does not own, its inputs and outputs, error behavior, dependencies, and security assumptions.

A contract should describe relevant behavior as well as data shapes. For example, an upload endpoint accepting a user identifier does not establish who checks whether the caller may upload for that user.

### Scope access to the task

Give AI development tools and runtime components only the repository, data, credentials, and permissions their work requires. Repository separation makes this practical only when access restrictions are actually enforced.

### Keep governance and runtime boundaries explicit

A repository provides a source-control boundary. A package provides a distribution boundary. A process, service, or sandbox may provide a runtime boundary.

Separate repositories do not automatically isolate runtime privileges, require network services, or require separate deployments. Modules can be libraries within one application.

### Preserve a view of the whole system

Maintain enough system context to understand dependencies, trust boundaries, data flows, and compatible versions without requiring every task to inspect every implementation.

The location, format, and ownership of this system view remain open.

### Use evidence to reconsider boundaries

A candidate boundary should make the responsibility easier to explain, verify, and audit.

Frequent coordinated changes, circular dependencies, extensive shared state, or the need to inspect many other implementations may indicate that a boundary needs revision. These are review signals rather than automatic reasons to merge or split repositories.
