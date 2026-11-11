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
