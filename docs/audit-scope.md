# Audit scope

**Agreed requirement:** auditing covers every aspect of a module, including data access and permissions.

The coverage below is an initial proposal for making that requirement concrete. The audit procedure, reviewer roles, release gates, and required evidence formats remain open.

## Coverage

| Area | Questions the audit should answer | Example evidence |
| --- | --- | --- |
| Purpose and requirements | Does behavior match the stated responsibility? What behavior is prohibited? | Scope, requirements, acceptance criteria |
| Architecture and boundaries | What does the module own? Where are its trust boundaries and assumptions? | Dependency map, data flows, boundary description |
| Implementation | Are normal, failure, concurrency, and resource-exhaustion paths handled correctly? | Code review, tests, analysis findings |
| Contracts | Are inputs, outputs, failures, side effects, and compatibility expectations explicit? | API or package contracts, contract tests |
| Authentication and authorization | How are identities established and actions authorized? Who enforces each check? | Authentication flows, authorization rules, negative tests |
| Data access and lifecycle | What data is read, created, changed, stored, logged, transmitted, retained, and deleted? | Data inventory, access rules, retention behavior |
| Permissions and isolation | What can development tools, build jobs, and runtime components access? Are boundaries enforced? | Repository permissions, IAM policies, sandbox and network configuration |
| Secrets and cryptography | How are secrets and keys issued, stored, used, rotated, and revoked? | Secret-handling design, redacted configuration, rotation evidence |
| Dependencies and supply chain | What direct and transitive dependencies are used, where do they come from, and what risks or licensing constraints apply? | Dependency inventory, lockfiles, provenance, vulnerability and license findings |
| Build and release | Can artifacts be traced to reviewed source? Who or what can modify and publish them? | Pipeline configuration, build records, artifact integrity and provenance |
| Deployment and configuration | Do deployed settings preserve the assumptions reviewed in source? | Infrastructure definitions, environment configuration, deployment records |
| Runtime behavior | What happens during abuse, failures, retries, recovery, and operational changes? | Runtime tests, limits, monitoring and recovery evidence |
| Observability and privacy | Are failures observable without exposing secrets or sensitive data? | Logging review, redaction tests, alerts, access and retention rules |
| Module interactions | Do consumers and providers agree about identity, ownership, validation, and failure handling? | Integration tests, system threat analysis, compatible version records |
| Maintenance | How are findings, dependency changes, incidents, and obsolete versions handled? | Finding records, remediation history, update and support policies |

Evidence should explain why any area is not applicable. An empty checkbox is not evidence.

## Audit conclusions and limits

A proposed audit record should identify:

- The source revision and artifact examined.
- Dependency versions and the relevant deployment and configuration.
- Scope, methods, reviewer, and supporting evidence.
- Findings, unresolved questions, and limitations.
- Assumptions about other modules and the environment.
- Changes that require the conclusion to be revisited.

An audit conclusion applies to its stated scope and conditions. It does not establish that all future versions or all deployments are safe.

## Verification across boundaries

For example, a login module may correctly issue an identity token while an upload module fails to enforce ownership of the destination. Reviewing token issuance alone would miss the system vulnerability.

Auditing therefore needs both module-level evidence and evidence that the assembled system enforces its security requirements.

## Decisions still open

- Which evidence is mandatory for every module, and how does review depth vary with risk?
- How are reviewers selected and kept sufficiently independent of the implementer?
- What findings block a release, and who can accept unresolved risks?
- Which code, dependency, contract, permission, configuration, or deployment changes require renewed review?
- How are audit conclusions and affected consumers updated when assumptions change?
