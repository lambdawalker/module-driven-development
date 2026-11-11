# Recovery planning guidelines

**Status: discussion draft, 2026-11-11.** These guidelines help shape the conversation between developer and AI before implementation and when new information emerges. Artifact examples are prompts, not universal rules or prescribed rollback procedures.

## Direction established in discussion

Recovery belongs in module planning. AI must raise the question **“What do we do if this fails?”** if humans do not.

Different artifacts and state changes require different recovery methods. The planning conversation should identify what matters about the change, what it affects, and how recovery would work in that specific context.

A named human accepts responsibility in advance for recovery of an artifact or coordinated group of artifacts. The person knows the planned change window, how they will be notified, and that they may need to act promptly. Recovery execution requires explicit human authorization.

The change window is scheduled; rollback is conditional on a problem rather than automatically scheduled for that date.

## Artifact-specific conversation prompts

The examples below are starting points to adapt to the project. They do not authorize any recovery action.

| Artifact or affected state | Questions to explore | Possible recovery approaches to evaluate |
| --- | --- | --- |
| Source code or built application | Which previous revision or build works? Is it compatible with current state? Can installed clients actually be reverted? | Revert source, restore a previous deployment, or ship a corrective version |
| Dependency declaration or library artifact | Which exact version was working? Which consumers changed? Is the previous artifact still available? | Restore the dependency selection, rebuild, and verify affected consumers |
| Images, text, colors, or other assets | Which content version must be restored? Are caches involved? Does the change affect accessibility or comprehension? | Restore versioned assets and verify delivery and user experience |
| Database schema | Will older code still work with the changed schema? Could reversal discard information? | A compatible schema transition, a tested reverse migration where safe, or a forward repair |
| Database records | What valid writes happened since the change? Can affected records be identified? | Targeted correction or carefully scoped restoration that accounts for newer valid data |
| Configuration or feature flags | Which values changed and which deployments use them? | Restore compatible settings or disable an affected feature if authorized |
| Infrastructure or routing | Which resources contain state? Would reverting recreate or destroy them? | Restore a known working configuration with an explicit state-preservation plan |
| Permissions or access policies | Which access was unintentionally granted or blocked? Would restoring the old policy reintroduce a vulnerability? | Restore intended access through a reviewed policy correction |
| External effects | Were messages sent, payments processed, or data disclosed? What cannot be undone? | Compensating or corrective action, reconciliation, and appropriate communication |

Recovery may require rollback, forward repair, compensation, or a combination. A successful command does not establish successful recovery.

### Database example

Restoring a backup may remove valid records written after the backup. Reverting application code may also fail if the current schema or data format no longer matches it.

Discuss the affected records and schema, newer valid writes, compatibility with the fallback application, and how the proposed recovery preserves or reconciles state. Identify any unavoidable loss or uncertainty for an explicit human decision. Select the procedure based on the actual change rather than assuming “restore backup” is sufficient.

### Accessibility example

A palette or text change can make an otherwise working application difficult to understand or use. Recovery verification should check the affected user experience, not merely that old files were copied successfully. Code, content, and configuration may need to be restored as a compatible combination.

## Suggested planning prompts

Use the relevant questions to build a proportionate recovery section in the module agreement:

- What is important about this change, and what must remain usable?
- Which artifacts, state, consumers, and teams could be affected?
- What signals indicate failure, and who will receive them?
- How can the impact be contained while awaiting a human decision?
- What previous artifacts, backups, access, and tools would recovery need?
- Who accepts responsibility, who authorizes recovery, and who executes it?
- What is the change window and observation period? Is the responsible person available?
- Is a backup contact or escalation route needed?
- Which actions, such as pausing a rollout, are authorized while recovery approval is pending?
- How will recovery preserve newer valid work and handle irreversible effects?
- What rehearsal or evidence is appropriate to establish feasibility?
- How will affected consumers and users verify that recovery succeeded?

These prompts guide discussion. The project-specific approved agreement determines the actual procedure, responsibilities, and authorization. The guidance itself does not grant permission to pause, modify, or restore a system.

## Course correction during development

Planning is revisitable. Discovering during development that a new database or an unanticipated database change is necessary is a normal reason to revisit the agreement, including in the middle of a sprint.

The developer and AI hold a course-correction planning conversation:

1. Describe the discovery, its motivation, and the difference from the approved agreement.
2. Reassess affected artifacts, recovery implications, acceptance criteria, and evidence.
3. Recheck dependencies and ownership, using the project graph when available; identify newly affected teams and bring them into the conversation through an authorized notification mechanism.
4. Revise the agreement and obtain the human approvals required by the updated impact and project policy.
5. Resume the affected implementation against the revised agreement.

The action that exceeds current authorization stays paused while the decision is pending. Independent work can continue within existing authorization. Mid-development discovery is not itself a failure of the methodology; silently treating a material discovery as already approved would defeat the agreement.

Retain the earlier agreement and the reason for revision so decisions and resulting evidence remain traceable. Neither an ongoing sprint nor earlier approval substitutes for approval of newly introduced scope.

## Relationship to project memory

The proposed [Project Memory Graph](project-memory-graph.md) can link affected artifacts and versions, recovery owners, revised agreements, pending decisions, and evidence. It can help find affected teams without replacing human responsibility or the actual recovery tools.

Related: [Specifications and approvals](specifications-and-approvals.md).
