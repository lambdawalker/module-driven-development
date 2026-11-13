# Topics needing more detail

**Status: open discussion follow-ups, 2026-11-11.** This list keeps unfinished design conversations visible. Section headings use stable names so cross-document links do not break as statuses evolve; items close only when outcomes are recorded in their target documents.

## Artifact-specific recovery guidance

**Status: requested follow-up — return to this conversation. Not yet complete.**

Revisit the recovery-planning discussion and systematically work through common artifact types, pre-authorized break-glass containment bounds, and full recovery procedures.

The [recovery planning guidelines](recovery-planning-guidelines.md) establish the split between pre-authorized reversible containment and human-authorized state recovery, alongside initial database, installed-client skew, and accessibility examples. Deeper worked patterns remain to be drafted:

- [ ] Agree on a practical catalog of artifact types used across mobile, Go backend, ML model, and infrastructure repos.
- [ ] Deepen the database playbook: expand-and-contract phases, isolating post-deploy valid writes, idempotent reconciliation, and verification.
- [ ] Develop worked playbooks for installed mobile clients, transitive library regressions, code-free ML model bundles, CDN/cached UI assets, IaC stateful resources, IAM policies, and irreversible external side effects.
- [ ] For each playbook, specify failure signals, safe break-glass containment vs. human-authorized repair, coordinated multi-artifact order of operations, and post-recovery verification checks.
- [ ] Define human readiness, change-window coverage, unreachable-owner escalation timeouts, and post-containment review records.
- [ ] Distill the resulting playbooks into sharper planning prompts for Change Agreements.

**Intended result:** adaptable, artifact-specific conversation playbooks with explicit limitations—keeping project-specific authority inside each approved Change Agreement.

## Artifact signals and scheduled checks

**Status: requested follow-up — return to this conversation. Not yet complete.**

Extend the artifact catalog with concrete invariants relied upon by consuming teams (such as S3 bucket public-access blocks, KMS rotation state, or API latency/error ceilings) and connect operational drift detection to project ownership.

Two signal phases complement each other:

- **Planning-time signals:** checking a proposed Change Agreement against repository dependency manifests / [Project Memory Graph](project-memory-graph.md) to bring dependent teams into review before code ships.
- **Operational state signals:** scheduled or event-driven checks verifying that live artifacts still satisfy required invariants, catching out-of-band console edits or configuration drift.

To keep [Project Memory Graph](project-memory-graph.md) focused on architectural memory rather than rebuilding observability infrastructure, operational checks should run in dedicated cloud/infrastructure monitors (such as AWS Config, CloudWatch, or Prometheus) and emit structured drift/alert events into the graph for impact routing. Check intervals (10 or 30 minutes were illustrative examples) depend on blast radius, detection-delay tolerance, and cost.

- [ ] Inventory concrete stateful artifacts from current projects (`go.attestra.aws` buckets, databases, IAM roles, exported Android configs) as worked examples.
- [ ] Map each artifact invariant to its native monitor check, dependent modules/teams, break-glass containment option, and human recovery owner.
- [ ] Design the event payload linking external monitor failures into Project Memory Graph obligations and notifications.
- [ ] Distinguish confirmed invariant violations from inconclusive runs, monitor permission errors, or dead-man-switch/stale monitor silence.
- [ ] Define monitor least-privilege read access and self-health verification.

### Additional signal sources to explore

Return to the question: **“What could change around an unchanged artifact that invalidates earlier acceptance?”**

- [ ] **Changed use or data sensitivity:** for example, storing identity documents in an S3 bucket originally audited for public thumbnails without changing bucket ACLs—requiring a Module Specification / Change Agreement review trigger.
- [ ] **Changed ownership:** team reorganization or maintainer departure requiring explicit re-acceptance of recovery and review duties.
- [ ] **User reports and incidents:** accessibility barriers, device-specific camera/overlay bugs, or unexpected denials exposing test gaps.
- [ ] **Surrounding-system advisories:** transitive CVEs, OS permission changes (Android/iOS SDK target updates), or upstream API deprecations invalidating a prior Release Attestation.
- [ ] **Verification degradation:** skipped suites, flaky-test quarantine abuse, or silent monitor failures; lost visibility must surface as degraded assurance rather than green status.
- [ ] **Behavioral analytics:** anomalous spikes or drops in completion rates, retries, abandonment, or duration—validated against instrumentation health, traffic mix, and rollout cohort before attributing to product regressions, routing to teams for investigation rather than automatic rollback.

## Signed production authorization

**Status: requested follow-up — return to this conversation. Concept only; mechanism not selected or implemented.**

Design a cryptographically verifiable human sign-off for promoting an integrated release candidate to production, bound directly to the candidate's **Release Attestation** (see [Specifications and approvals](specifications-and-approvals.md#library-artifact-publication-versus-human-qa-before-production-release)).

This internal governance signature proves *which named human authorized production promotion after QA* and *which exact artifact digest was authorized*; it complements rather than replaces Google Play App Signing or Apple distribution certificates.

To avoid inventing custom cryptography or hitting hardware-token payload limits, evaluate established standards first:

- [ ] **Candidate attestation format:** evaluate an in-toto statement with an appropriate human-authorization predicate, linked to build provenance where available (binding source commit SHA, immutable artifact SHA-256 digest, resolved BOM/dependency digest, target environment, QA summary hash, timestamp, and authorization scope) rather than raw multi-megabyte binaries.
- [ ] **Hardware security key path:** evaluate **OpenSSH FIDO2 resident keys** with tested user-presence/user-verification policy and compatible signing tooling and **Sigstore / Cosign** hardware-token signing.
- [ ] **Mobile phone path:** evaluate **WebAuthn / Passkey assertions** signing a challenge derived from the canonical Release Attestation SHA-256 digest, verifying user-verification (`uv`) flags and credential binding.
- [ ] **Verification and transparency log:** define how deployment pipelines verify the signature against an enrolled maintainer public-key registry (protected outside AI agent write scope) and append records to an append-only audit log before releasing credentials to deploy.
- [ ] **Key lifecycle:** specify enrollment ceremony, backup key requirements for solo developers, lost/compromised key revocation without invalidating historical pre-revocation attestations, and separation between QA evidence quality and cryptographic proof of sign-off.

## Other open topics already identified

The three-record model and verification responsibilities are documented. Risk tiers, staging tooling, and Git-native manifests remain project-policy or reference proposals requiring practical validation. Concrete follow-ups remain:

- [ ] **Governance file schemas:** finalize exact YAML/Markdown schemas for `MODULE.md` (Module Specification), Change Agreements, and Release Attestations.
- [ ] **Remote approval runner trial:** test GitHub App/bot identity setup, enforced branch rulesets, evidence and inspection-capability routing, commit-SHA approval binding, revision invalidation, and idempotent runner resume; record trial notes in [Specifications and approvals](specifications-and-approvals.md).
- [ ] **Context-isolated verifier harness:** prototype the clean-room CI/agent invocation that feeds only `MODULE.md`, Change Agreement, and public interfaces to the adversarial verifier while enforcing protected-baseline guards on locked test files; also prototype source/security review with implementation access.
- [ ] **Cross-repo inner-loop & BOM tooling:** extend [Repository automation](repository-automation.md) (`gradle.android-repo-setup` and Go composition tooling) with commit-SHA staging feeds, `binary-compatibility-validator` gates, BOM alignment, and ruleset drift checks.
- [ ] **Phase 1 Git-native graph prototype:** test CI manifest aggregation, revision-vector freshness, conflicting plans, and read-only MCP query usefulness against real repositories; compare with the dedicated-service option; see [Project Memory Graph](project-memory-graph.md).
- [ ] **Boundary graduation and lifecycle rules:** define practical metrics for graduating day-0 subprojects into standalone repos, enforcing N-back client deprecation windows, and retiring obsolete modules.

## How to close a follow-up

Record the agreed design or trial evidence in the canonical methodology document, preserve any residual limitations, and link the commit or decision record here.
