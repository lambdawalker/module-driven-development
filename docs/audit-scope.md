# Audit scope

**Agreed requirement:** auditing covers every aspect of a module, including data access, permissions, AI-specific threat surfaces, and cross-module interactions, scaled proportionally to module risk.

The coverage and risk-tier model below make that requirement actionable while avoiding review fatigue on low-risk leaf libraries. Exact evidence schemas and automated policy thresholds remain open.

## Coverage

| Area | Questions the audit should answer | Example evidence |
| --- | --- | --- |
| Purpose and requirements | Does behavior match the stated Module Specification? What capabilities are explicitly prohibited? | Module Specification, Change Agreements, negative tests |
| Architecture and boundaries | What does the module own? Where are its trust boundaries and upstream/downstream assumptions? | Dependency map, data-flow diagram, boundary specification |
| AI agent security and prompt injection | Can untrusted issues, PR text, dependency files, build logs, or MCP/graph records manipulate agents? Are agent tools, filesystem scope, network egress, and credentials isolated? | Agent sandbox/egress rules, untrusted-input separation, package registry allowlists, credential-exclusion checks |
| Implementation | Are normal, edge, concurrency, and failure paths handled without correlated test blind spots? | Code review, context-isolated adversarial/property tests, static analysis |
| Contracts | Are inputs, outputs, units, error semantics, side effects, and caller obligations explicit and enforced? | Public API/ABI dumps, schema definitions, locked contract tests |
| Cross-boundary version skew | Do backend endpoints, exported configs, and wire schemas preserve N-back compatibility with older installed clients that cannot be reverted? | Wire-compatibility checks, expand-and-contract migration plan, client deprecation policy |
| Authentication and authorization | How are identities established and actions authorized? Which module enforces each check? | Auth flows, authorization rules, cross-module negative tests |
| Data access and lifecycle | What data is read, created, mutated, logged, transmitted, retained, and deleted? Did data sensitivity change? | Data inventory, access rules, retention/deletion verification |
| Permissions and isolation | What can development agents, CI jobs, and runtime components access? Are repository rulesets and sandboxes enforced? | Branch protection rulesets, IAM policies, container/runner network configs |
| Secrets and cryptography | How are keys and secrets issued, stored, rotated, and revoked? Are signing keys strictly unreachable by AI agents? | Secret architecture, KMS/OIDC release isolation, rotation records |
| Dependencies and supply chain | Are dependencies encapsulated (`implementation` vs. `api`), aligned with ecosystem BOMs, pinned, and free of hallucinated or vulnerable packages? | Lockfiles, SBOM/provenance, transitive exposure check, vulnerability/license scans |
| Build and release | Can published artifacts be traced cryptographically to reviewed source and Release Attestations? | Reproducible build config, SLSA/in-toto provenance, release guard logs |
| Deployment and configuration | Do live settings match source assumptions and preserve safe rollback/containment controls? | IaC diffs, exported client config checks, feature-flag/kill-switch verification |
| Resource budgets and resilience | Does the module stay within memory, binary-size, thread-pool, latency, and retry/backoff budgets without causing retry storms? | Benchmark/size gates, executor ownership review, backoff/circuit-breaker tests |
| Runtime behavior, observability, and privacy | Are failures and drift detectable without leaking secrets or PII in logs or analytics? | Redaction tests, structured alert signals, recovery drill evidence |
| Module interactions | Do composed modules agree on identity, validation, ownership, version ranges, and failure semantics? | Consumer-led obligation tests, system threat model, verified version matrix |
| Maintenance | How are advisories, ownership transfers, stale assumptions, and deprecated versions handled? | Remediation records, active owner confirmation, support sunset policy |

Evidence must explain why any domain is inapplicable (for example, zero network or storage capability in a pure math library). An empty checkbox is not evidence.

## Risk tiers and delta-scoped audits

Applying identical manual audit ceremony to a stateless geometry helper and an authentication backend causes reviewer fatigue and weakens scrutiny where it matters most. Modules declare one of three risk tiers in their **Module Specification**:

| Tier | Profile and examples | Baseline and ongoing audit expectations |
| --- | --- | --- |
| **Tier 1 — Critical trust & shared tooling** | Auth, cryptography, key attestation, permissions, IAM/Pulumi composition, payment/contract signing, and shared CI/release plugins (`gradle.android-repo-setup`) | Full human-led baseline audit across all domains, workstation-only review for boundary/code changes, context-isolated adversarial testing, and mandatory delta audit on any security, dependency, or contract touch. |
| **Tier 2 — Stateful, networked, or UI feature modules** | Feature backends, storage/S3 adapters, camera/ML integration libraries (`Card Detector`) | Targeted baseline audit focused on contracts, permissions, data lifecycle, version skew, and resource budgets; automated CI gates for routine internal changes; delta human review when deterministic escalation gates fire. |
| **Tier 3 — Pure-compute deterministic leaf modules** | Stateless, zero-I/O, zero-permission libraries with minimal or zero dependencies (`Coordinates`) | Lightweight boundary verification proving absence of I/O, network, and permissions, paired with strong mathematical/invariant suites authored by a context-isolated verifier and automated ABI/dependency gates. |

### Baseline audits versus delta re-audits

A **baseline audit** establishes initial trust for a module revision and its declared tier. Subsequent changes within unchanged boundaries rely on automated deterministic gates and context-isolated verification unless a **re-audit trigger** occurs:

- Elevating module capabilities, permissions, network egress, or risk tier.
- Modifying `Tier 1` security logic, cryptographic primitives, or shared release automation.
- Introducing new external dependencies or receiving a relevant supply-chain/vulnerability advisory.
- Changing data sensitivity (for example, storing identity documents in a bucket previously audited for public thumbnails).
- Breaking wire or binary compatibility across module boundaries.

## Audit conclusions and limits

A proposed audit record (linked from the module's **Release Attestation**) should identify:

- Examined source commit SHA, artifact digest, risk tier, and audit type (baseline vs. delta).
- Resolved dependency lock state, build tooling version, and target deployment assumptions.
- Scope, methods, human/verifier identities, and supporting evidence links.
- Findings, accepted residual risks, and explicit assumptions about upstream or downstream modules.
- Specific conditions or triggers that invalidate the conclusion.

An audit conclusion applies strictly to its stated revision, configuration, and environmental assumptions.

## Verification across boundaries

A login module may validly issue an identity token while an upload module fails to verify destination ownership, or three modules with individually polite retry loops may multiply into a 27x retry storm when chained.

System auditing at the **Application Composition Host** level must therefore verify cross-module authorization enforcement, N-back client-backend skew, and aggregate resource/retry behavior across the assembled graph.

## Decisions still open

- Exact machine-readable schema for linking audit records into Release Attestations.
- Rules for qualifying independent human or specialized AI security auditors on `Tier 1` modules.
- SLA and quarantine policies when a post-release advisory invalidates an active module attestation.
