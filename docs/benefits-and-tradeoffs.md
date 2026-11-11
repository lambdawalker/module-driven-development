# Benefits and tradeoffs

These are expected effects of the proposed approach, not measured results or security guarantees.

## Benefits

| Benefit | Why it matters with AI doing most development |
| --- | --- |
| Focused context | A bounded responsibility reduces the unrelated information an AI must consider. |
| Contained audits | Reviewers can examine one implementation against explicit responsibilities, contracts, and security expectations. |
| Restricted development access | Separate repositories allow task-specific access when tooling and credentials enforce it. |
| Reviewable changes | Changes within a clear scope can be easier to explain, inspect, test, and revert. |
| Clear acceptance criteria | A bounded unit can specify expected behavior, failure cases, and prohibited behavior. |
| Independent release management | Modules can be fixed and versioned separately when compatibility permits. |
| Reuse | A module and its verification evidence may support multiple applications, subject to their different assumptions and environments. |
| Replaceability | A stable contract can permit implementation changes without requiring consumer changes. |
| Traceability | Requirements, decisions, changes, audit findings, and releases can be tracked per module. |

## Costs and failure modes

| Cost or risk | Consequence |
| --- | --- |
| Fragmented understanding | A locally correct change can violate assumptions elsewhere in the system. |
| Vulnerabilities between modules | Authorization, validation, or ownership checks may be omitted because each module assumes another performs them. |
| Contract maintenance | APIs, schemas, failures, security assumptions, and compatibility rules need deliberate ownership and verification. |
| Coordinated changes | One feature change may require updates and releases across several repositories. |
| Version proliferation | Consumers need a way to select and verify compatible module combinations. |
| Repeated infrastructure | Builds, publishing, dependency maintenance, and access management must be maintained across repositories. |
| Additional supply-chain surfaces | More artifacts, credentials, and pipelines create additional places to protect. |
| Excessive fragmentation | Communication and coordination can outweigh the value of a very small unit. |
| Correlated AI mistakes | An AI can write implementation and tests around the same incorrect assumption. |
| Stale audit conclusions | Changes to dependencies, configuration, deployment, or consumers can invalidate earlier findings. |

## Implications for the methodology

Repository separation needs explicit contracts, enforced permissions, integration verification, and a maintained view of the system.

Verification needs evidence beyond an implementer's assertion that work is correct. Passing tests alone cannot establish that the requirements, security assumptions, or tests themselves are adequate.

The workflow for achieving this remains to be defined.
