# Benefits and tradeoffs

These are expected effects of the proposed approach, not measured results or security guarantees.

## Benefits

| Benefit | Why it matters with AI doing most development |
| --- | --- |
| Focused context | A bounded responsibility reduces the unrelated information an AI must consider. |
| Contained audits | Reviewers can examine one implementation against explicit responsibilities, contracts, risk tiers, and security expectations. |
| Restricted development access | Separate repositories allow task-specific access and egress limits when tooling and credentials enforce them. |
| Reviewable changes | Changes within a clear scope can be easier to explain, inspect, test, and revert. |
| Clear acceptance criteria | A bounded unit can specify expected behavior, invariants, failure cases, and prohibited behavior. |
| Independent release management | Modules can be fixed and versioned separately when binary and wire compatibility permit. |
| Reuse | A module and its verification evidence may support multiple applications, subject to their different assumptions and environments. |
| Replaceability | A stable contract can permit implementation changes without requiring consumer changes. |
| Traceability | Specifications, change agreements, audit findings, and release attestations can be tracked per module. |

## Costs and failure modes

| Cost or risk | Consequence |
| --- | --- |
| Fragmented understanding | A locally correct change can violate assumptions elsewhere in the system. |
| Vulnerabilities between modules | Authorization, validation, or ownership checks may be omitted because each module assumes another performs them. |
| Contract maintenance | APIs, schemas, failures, security assumptions, and compatibility rules need deliberate ownership and verification. |
| Cross-boundary version skew | Installed mobile clients and live backends update asynchronously; unversioned wire changes break older clients that cannot be rolled back. |
| Coordinated changes | One feature change may require staged updates and verification across several repositories before release. |
| Version proliferation and diamond conflicts | Consumers need resolved-version verification, appropriate dependency encapsulation, and compatibility checks; BOM alignment is one option to avoid conflicting transitive versions. |
| Aggregate resource exhaustion | Locally valid modules can duplicate thread pools, blow mobile binary or frame-time budgets, or trigger cascading retry storms when composed. |
| Premature day-0 fragmentation | Splitting unsettled greenfield features into separate repositories too early multiplies cross-repo friction during initial discovery. |
| Repeated infrastructure | Builds, publishing, dependency maintenance, and access management must be maintained across repositories via shared tooling. |
| Additional supply-chain surfaces | More artifacts, credentials, and pipelines create additional places to protect. |
| Correlated AI mistakes | An implementing AI can write both code and unit tests around the same flawed interpretation of requirements. |
| Indirect prompt injection and agent overreach | Untrusted issues, dependency files, or shared memory notes can steer an agent if merge gates and permissions rely on agent judgment. |
| Stale audit conclusions | Changes to dependencies, configuration, data sensitivity, deployment, or consumers can invalidate earlier findings. |

## Implications for the methodology

Repository separation requires explicit contracts, enforced least-privilege sandboxes, N-back wire compatibility rules, pre-release cross-repo staging, system-level resource budgets, and a maintained view of the assembled system.

Verification requires structural independence from implementation. Because passing self-authored tests cannot establish that requirements, security boundaries, or tests themselves are adequate, the methodology pairs **deterministic CI policy gates** (which remove merge authority from AI self-classification) with **context-isolated verification** (where behavioral tests are derived independently from the approved contract and specification), alongside source/security review and human QA before production promotion. No single verification role establishes complete assurance.
