# Topics needing more detail

**Status: open discussion follow-ups, 2026-11-11.** This list keeps unfinished topics visible for future planning conversations. Items remain open until discussed and their outcomes recorded; an outline or example is not completion.

## Artifact-specific recovery guidance — return to this conversation

**Requested follow-up. Not yet complete.**

Revisit the recovery-planning discussion and systematically work through common artifact types and guidelines for handling their failure and recovery.

The [recovery planning guidelines](recovery-planning-guidelines.md) contain an initial artifact table and a useful database example, including the risk of losing newer valid writes when restoring a backup. These are a starting point, not a complete artifact catalog or a set of validated recovery procedures.

- [ ] Agree on a practical list of common artifact types.
- [ ] Deepen the database example: schema versus records, compatibility, preserving newer valid work, and verification.
- [ ] Develop comparable examples for code/builds, dependency versions, assets/content (including accessibility), configuration, infrastructure, permissions, and external effects.
- [ ] For each type, discuss failure signals, feasible rollback or repair, irreversible effects, coordinated recovery with other artifacts, and evidence that recovery worked.
- [ ] Discuss human ownership, readiness, notifications, authorization, and escalation in those examples.
- [ ] Use the examples to improve the questions AI raises before implementation and during course-correction planning.

**Intended result:** adaptable, artifact-specific conversation guidance with worked examples and explicit limitations. Keep these as guidelines, not universal rules; each project's module agreement determines its actual recovery plan.

## Artifact signals and scheduled checks — return to this conversation

**Requested follow-up. Not yet complete.**

Start with artifacts actually used in the current project, such as databases, S3 buckets, and their configuration. Extend the artifact catalog above with the conditions each consuming team relies on and the signals that reveal whether those conditions still hold.

The discussion identified two complementary signal sources:

- **During planning:** a proposed artifact-state change is checked against the graph, revealing dependent teams that need to join the conversation.
- **After release / during operation:** a separate scheduled process checks the actual state of an important artifact, helping detect mistakes or drift even when planning did not prevent them.

A consuming team could request periodic checks of a condition such as private versus public access. Intervals of **10 or 30 minutes were examples**, not chosen schedules or universal defaults. This capability could belong to the software that includes Project Memory Graph; implementation and ownership remain to be designed.

- [ ] Inventory relevant artifacts from the current project as the starting point for worked examples.
- [ ] For each artifact, connect the required condition to its signal, verification method, dependent teams, and responsible human.
- [ ] Explore scheduled artifact-state checks (“heartbeat” checks) alongside planning signals and change events.
- [ ] Choose check intervals according to consequences, acceptable detection delay, and operational cost.
- [ ] Define how observations and evidence link into the graph and trigger notifications or review for affected teams.
- [ ] Distinguish a failed condition from an inconclusive check, insufficient permissions, or a missed/stale monitoring run.
- [ ] Discuss monitor ownership, access, alert handling, and how to know that the monitoring process itself is working.
- [ ] Link confirmed problems to the agreed human-authorized recovery process.
- [ ] Evaluate which parts belong in the graph companion system and which integrate external monitoring.

**Intended result:** adaptable guidance for planning and operational signals by artifact type, with concrete project examples. No monitoring job, notification integration, or schedule has been configured by recording this follow-up.

### Additional signal sources to explore

Return to the question: **“What could change around this artifact that would make our earlier acceptance no longer sufficient?”** These are candidate signals to investigate, not a finalized trigger policy.

- [ ] **Changed use or data sensitivity:** for example, a private bucket starts holding identity documents instead of public product photos without a technical configuration change. Explore how teams declare new uses and when manual review is needed.
- [ ] **Changed ownership:** a team reorganizes, a responsible person leaves, or responsibility transfers. Verify that someone accepts the existing obligations, access, and response duties.
- [ ] **User reports and incidents:** accessibility complaints, unexpected access denials, corrupted output, and other reports may expose gaps in automated checks. Discuss triage and evidence.
- [ ] **Surrounding-system changes:** dependency vulnerability advisories, external API behavior, and configuration changes may invalidate assumptions about an unchanged artifact.
- [ ] **Changes to verification:** disabled checks, altered expected results, checks that stop running, and missing evidence should prompt investigation. Lost visibility is not evidence of a healthy artifact.
- [ ] **Behavioral analytics:** explore unexpected increases or decreases in feature usage, completion rates, abandonment, retries, and time to complete as indirect signals from users. Compare with relevant traffic, historical patterns, rollout versions, and affected user groups; a usage spike may reflect retries rather than success, and a drop may reflect lower traffic rather than a broken feature. Check analytics instrumentation and delivery before attributing a change to the product. Link the signal through the graph to responsible teams for investigation, not automatic rollback. Decide appropriate baselines, observation windows, data-access/privacy constraints, and response thresholds during project planning.
- [ ] **Graph-based routing:** connect each signal to affected conditions, consumers, and teams; define who investigates, how they are notified, and which evidence supports a decision.

## Signed production authorization — return to this conversation

**Requested follow-up. Concept only; mechanism not selected or implemented.**

Explore an internal production-promotion authorization signed by a named human using a mobile phone or a hardware security key (for example, a USB security key). The intent is a verifiable internal record that identifies who authorized promotion through which registered credential and binds that authorization to the artifact being promoted.

This serves the human QA release decision; it is distinct from Google Play or Apple distribution signing and does not replace those platform requirements.

- [ ] Explore phone-based and hardware-key approaches and their actual signing capabilities; do not assume every authentication key can sign arbitrary artifacts.
- [ ] Define whether the signature covers the artifact directly or an authorization statement containing its digest, tested build/version, destination environment, QA decision, and authorization scope.
- [ ] Design the internal authorization log and how promotion verifies the record against the exact candidate.
- [ ] Discuss credential enrollment, human identity, loss/revocation, and verification of historical records.
- [ ] Distinguish proof of authorization from the QA evidence itself; a signature does not prove that testing was sufficient.

Keep this as a note for a later design conversation rather than a chosen cryptographic architecture.

## Other open topics already identified

These entries summarize remaining questions in the existing draft; they do not reopen decisions already established.

- [ ] **Remote approval trial:** test notification delivery, approval, rejection, revisions, no response, and runner interruption/resume; record observations before removing the provisional marker. See [specifications and approvals](specifications-and-approvals.md).
- [ ] **Verification and audit detail:** refine evidence formats, independent verification, and what changes require another audit. Preserve the agreed automatic-acceptance and human-review paths. See [audit scope](audit-scope.md).
- [ ] **Project-specific policy refinement:** use practical cases to refine meaningful-change classification and approved automation boundaries. Keep the common guidance lightweight.
- [ ] **Integration and release automation:** detail artifact identity, published-artifact verification, reliable triggers, waiting consumer branches, and reruns after relevant changes. This implements the existing planning/acceptance model rather than introducing a separate approval process.
- [ ] **Project Memory Graph design and trial:** refine provenance, freshness, ownership, version combinations, synchronization, and access controls; test whether the proposed system improves continuity and impact analysis. See [concept document](project-memory-graph.md).
- [ ] **Maintenance and evolving boundaries:** discuss ongoing ownership, compatible versions, and signals for reconsidering module boundaries.

## How to close a follow-up

Record the conversation outcome in the relevant document, retain any unresolved questions, and link the resulting decision or trial notes here. No date or reminder has been scheduled by this list.
