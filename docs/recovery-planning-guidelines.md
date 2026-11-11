# Recovery planning guidelines

**Status: discussion draft, 2026-11-11.** These guidelines shape the planning conversation between developer and AI before implementation and during mid-sprint course corrections. Artifact examples are adaptable prompts, not universal rollback scripts.

## Direction established in discussion

Recovery belongs in module planning. AI must raise **“What do we do if this fails?”** whenever humans do not.

Different artifacts and state mutations require different recovery strategies. Planning identifies what could break, which consumers or state are affected, what immediate containment is safe, and how full recovery works in context.

A named human accepts responsibility in advance for recovery of an artifact or coordinated change group, knows the planned change window and notification channel, and is prepared to act. Executing state recovery or destructive rollback requires explicit human authorization.

The change window is scheduled; rollback is conditional on observed failure rather than an automatic timer.

## Break-glass containment versus full recovery

Requiring human sign-off before mutating state or restoring backups prevents panicked, destructive rollbacks—but waiting on an unreachable human during an active data leak or runaway error loop can worsen harm. The methodology therefore separates two distinct operational stages inside the **Change Agreement**:

| Stage | Scope and examples | Authorization model |
| --- | --- | --- |
| **Pre-authorized break-glass containment** | Bounded, non-destructive, reversible actions explicitly pre-approved during planning: disabling a feature flag / kill switch, halting an active progressive rollout, tightening rate limits, isolating an agent runner, or revoking an ephemeral session/API token upon confirmed breach criteria. | May be triggered immediately by an automated monitor or on-call responder within the pre-approved bounds defined in the Change Agreement, paired with immediate human paging and a mandatory post-containment review record. |
| **Full state recovery or repair** | Actions that alter persistent state, redeploy code, downgrade schemas, reconcile database writes, rotate root keys, or send external customer communications. | Requires explicit authorization from the responsible human (or designated escalation contact) after inspecting containment status and state impact. |

If no break-glass containment action was pre-authorized in the Change Agreement, automated systems may alert and block *further* rollout progression, but may not mutate live infrastructure or data unilaterally.

## Artifact-specific conversation prompts

The examples below are starting points to adapt per Change Agreement. They do not themselves authorize operational actions.

| Artifact or affected state | Questions to explore | Possible containment and recovery approaches |
| --- | --- | --- |
| Source code or backend service | Which previous revision works with current data? Can traffic shift back cleanly? | Pause rollout; redeploy known-good digest or ship forward fix |
| Installed mobile or desktop client | Shipped binaries on user devices cannot be force-reverted—what server kill switches or N-back API paths exist? | Toggle remote feature kill switch; preserve N-back backend endpoints; enforce minimum-client gate for critical security bugs; submit expedited store hotfix |
| Dependency declaration or library artifact | Which exact version worked? Did transitive types leak? Is the prior artifact immutable and cached? | Pin prior version in consumers, rebuild, and publish corrective patch release (never overwrite published versions) |
| Images, text, colors, or UI assets | Which asset bundle version restores clarity? Are CDN or client caches holding broken assets? Does it impair accessibility? | Purge/re-point versioned asset manifest; verify contrast, layout, and screen-reader usability |
| Database schema | Is the migration expand-and-contract compatible with the previous app build? Would reversal drop columns containing new data? | Use backward-compatible additive migrations first; prefer forward schema repair over destructive down-migrations |
| Database records | Which valid writes arrived after deployment? Can corrupted rows be isolated by timestamp, version tag, or tenant? | Halt offending writer via flag; run targeted idempotent reconciliation rather than blind full-snapshot overwrite |
| Configuration or feature flags | Which deployments consume the setting, and how fast do clients refresh config? | Revert flag/config version; verify client polling/cache TTL behavior |
| Infrastructure or routing | Which resources hold state (buckets, queues, KMS keys, databases)? Would IaC revert destroy them? | Protect stateful resources with deletion guards; restore routing/compute config with explicit state preservation |
| Permissions or access policies | Was overly broad access granted, or valid traffic locked out? Would reverting reopen a prior flaw? | Apply narrow deny/containment rule immediately; deploy reviewed least-privilege policy fix |
| External effects | Were emails/SMS sent, webhooks fired, cards charged, or secrets exposed? | Halt outbound queue; issue compensating transactions, credential revocations, and human-approved disclosures |

A green CLI exit code does not prove recovery succeeded; verify user-facing and consumer-facing behavior afterward.

### Database example

Restoring a full backup overwrites valid user transactions recorded after the snapshot, and rolling back application code fails if the live schema dropped or renamed expected columns.

Plan schema changes using expand-and-contract steps so the previous application binary remains compatible during the rollout window. Document how post-deploy valid writes are identified and reconciled, and require an explicit human decision whenever recovery entails data loss or ambiguity.

### Installed client and backend skew example

Unlike a backend container, an Android APK or iOS build installed on thousands of phones cannot be rolled back once distributed.

Recovery planning for mobile-facing changes must ensure that backend endpoints maintain N-back wire compatibility during rollout, remote feature flags can disable faulty new client flows cleanly, and emergency minimum-version enforcement exists for severe security regressions without bricking unaffected app functions.

### Accessibility example

A palette, contrast, or copy change can pass every HTTP 200 check while leaving users unable to complete authentication or read critical warnings. Recovery verification must validate actual user comprehension and assistive-technology flows across matching code, asset, and config versions.

## Suggested planning prompts

Use relevant prompts to write a proportionate recovery subsection in the Change Agreement:

- What user capability, security invariant, or data integrity property must remain intact?
- Which downstream consumers, installed client versions, and teams are in the blast radius?
- Which concrete signals (errors, latency, monitor checks, behavioral drop-offs) indicate failure, and who receives them?
- Which **break-glass containment** actions (such as flipping a kill switch or pausing rollout accrual) are pre-authorized while awaiting human recovery sign-off?
- What prior artifact digests, schema compatibility guarantees, backups, and credentials are needed for full recovery?
- Who is the named recovery owner during the change window, and who is the backup escalation contact if they are unreachable?
- How will recovery preserve valid concurrent writes and address irreversible external side effects?
- What dry-run, staging drill, or automated test demonstrates that the recovery path actually works?
- Which checks confirm end-to-end recovery for users and downstream modules?

## Course correction during development

Discovering mid-implementation (including mid-sprint) that a task requires an unplanned database change, new permission, or altered cross-module contract is normal engineering discovery—not a methodology failure, provided it is surfaced immediately rather than hidden.

Developer and AI hold a course-correction conversation:

1. State the discovery, why the approved Change Agreement cannot satisfy the goal as written, and whether the durable Module Specification is affected.
2. Re-evaluate blast radius, version skew, break-glass containment, recovery steps, and verification cases.
3. Check consumer ownership (via repository manifests or Project Memory Graph) and invite newly affected teams through an authorized channel.
4. Approve the revised Change Agreement (and Module Specification diff, if applicable) under project review rules.
5. Resume blocked work against the new revision while preserving prior agreement history for traceability.

Orthogonal work within existing authorization may continue while the revision awaits approval.

## Relationship to project memory

[Project Memory Graph](project-memory-graph.md) can map affected consumers, recovery owners, active change windows, external monitor alerts, and revised Change Agreements without replacing human authority or native cloud monitoring and deployment tools.

Related: [Specifications and approvals](specifications-and-approvals.md).
