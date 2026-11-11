# Repository automation and delivery

## Direction established in discussion

Independent repositories need shared automation for recurring setup and consistent delivery. The Android standard is being implemented through [gradle.android-repo-setup](https://github.com/lambdawalker/gradle.android-repo-setup); adoption is in progress.

A shared tool is itself a `Tier 1` critical-trust module with a version, consumers, maintenance responsibilities, and full security audit scope.

## Observed Android implementation

The studied tooling divides responsibilities between a Gradle plugin and a standalone Rust CLI.

| Responsibility | Implementation observed |
| --- | --- |
| Build and publication configuration | The library plugin configures release AARs, sources/documentation jars, reproducible jar settings, and a local verification repository. |
| Repository identity and customization | Checked-in configuration identifies the repository, library module, demo app module, tooling reference, and optional verification or workflow overrides. |
| Setup | The CLI supports APK signing, Maven signing key setup, GitHub environments, and encrypted local setup storage. |
| Workflow generation | Shared templates generate publication, tests, documentation, screenshots, signed demo APK, and release recovery workflows. |
| Drift detection | Check mode reports differences without writing; generation distinguishes local changes from template changes and requires an explicit overwrite mode for differing existing files. |
| Tooling selection | Workflow configuration accepts a release tag or full commit SHA, rather than a moving branch. A tag pin still depends on tag integrity. |
| Local artifact verification | The CLI checks the candidate publication, including coordinates, metadata, expected package/API characteristics, and artifact hashes. |
| Release identity | Publication reserves a version against source and artifact hashes before upload; the Gradle guard checks the reservation. |
| Public confirmation | Finalization verifies public artifacts; successful execution of an upload step alone does not establish a confirmed release. |
| Recovery | Pending and uncertain upload state is preserved for investigation; failed JitPack attempts have a separate reconciliation process. |

These observations come from source and documentation, not from executing setup or publishing an artifact.

### Adoption observed on 2026-11-11

| Repository | Observed state |
| --- | --- |
| Android Demo Common | Applies the setup library plugin; workflow configuration selects `app` and pins CLI tooling to `v0.1.6`. |
| Android Permissions | Applies the setup library plugin; configuration pins CLI tooling to a full commit and supplies module-specific verification and workflow commands. |
| Android Auth | Still has its earlier Vanniktech configuration and Python setup instructions in the inspected source. Shared-tooling migration is not complete there. |
| Coordinates | A Kotlin/JVM JAR library, rather than an Android AAR. Its delivery standard must respect that artifact type. |

This is a point-in-time study of selected repositories, not an ecosystem-wide adoption inventory. It does not establish the outcome of any release run.

## Backend composition and delivery

The application repository composes separately owned feature infrastructure through `go.infra.composer` and native Pulumi factories.

Feature modules supply declarations, contracts, runtime artifacts, and infrastructure entry points. The application supplies shared resource bindings, selects the features, validates configuration, and coordinates deployment.

The deployment tooling can export public Android configuration for the Android build. This connects backend outputs to the consuming client without placing deployment credentials in that configuration.

A published Android library and a deployed backend are different deliverables. Both need traceable versions and verification, but they use different tools and have different release gates (library verification gates versus deployable Human QA sign-off).

## Pre-release inner loop across repositories

When a change spans multiple repositories (for example, `Coordinates` → `Card Detector` → `Auth`), cutting permanent public release tags just to test whether a downstream consumer branch compiles and passes obligation checks creates version spam and broken intermediate releases. Shared tooling should support a three-stage progression:

1. **Local composite inner loop:** developers and local agents use build-system composition (such as Gradle `includeBuild`, Go `go.work` workspaces, or local build-scoped Maven repositories) to iterate across adjacent modules without mutating committed release coordinates.
2. **CI cross-repo staging loop:** pull requests or integration branches publish immutable, commit-SHA-versioned candidate artifacts (for example, `0.3.0-sha.<commit>`) to an internal or ephemeral staging registry. Waiting consumer integration branches resolve that exact candidate digest to verify consumer obligations before any public version is reserved.
3. **Public release promotion guard:** merging to `main` and cutting a public release strictly rejects local path overrides, `SNAPSHOT` coordinates, or ephemeral staging dependencies—requiring confirmed public dependency versions and immutable hashes.

## Dependency and transitive version governance

Independent module release cycles quickly trigger diamond-dependency conflicts if transitive boundaries are leaky (as seen when `Auth` manually unpacked model assets from an AAR to dodge an older transitive core library):

- **Internalize dependencies by default:** library modules must declare dependencies as `implementation` (or private internal imports) unless a dependency's types intentionally form part of the module's public contract (`api`).
- **Separate data/ML assets from runtime code:** package ML weights, schemas, and static calibration assets as code-free asset bundles or raw artifacts with versioned metadata manifests rather than embedding them inside AARs/JARs coupled to older runtime classes.
- **Align shared foundations via BOMs and ABI checks:** maintain an ecosystem Bill of Materials (or shared version platform) for cross-cutting runtime libraries, and enforce automated binary compatibility checks (such as Kotlin `binary-compatibility-validator` and wire-schema linters) in CI on every pull request.

## Credential isolation and repository protection

Because shared automation provisions signing keys, GitHub environments, and local encrypted setup files, strict privilege separation between AI agents and release infrastructure is mandatory:

- **Zero agent access to signing and production secrets:** AI coding, planning, and verification agents must never have filesystem read access, environment variable injection, or decryption passphrases for APK keystores, Maven GPG keys, `.project/setup` encrypted secrets, or cloud deployment credentials.
- **Isolated release execution:** artifact signing, public publication, and infrastructure deployment run exclusively inside protected CI environments (using hardware-backed keys or short-lived OIDC tokens) or human-invoked release enclaves after required approvals and status checks pass.
- **Enforced repository rulesets:** shared setup tooling should configure and drift-check branch protection on `main` and release tags—blocking direct pushes, requiring deterministic CI gates, dismissing stale reviews on new commits, pinning shared tooling by immutable commit SHA (or verified signed tag), and preventing bot identities from bypassing review rules.

## Proposed methodology rules

1. Maintain shared setup and delivery behavior in versioned, `Tier 1`-audited tooling instead of reproducing scripts per repository.
2. Keep repository-specific identity, artifact profile (AAR, Kotlin/JVM JAR, code-free model bundle, Go module), resource budgets, and verification commands in reviewable configuration.
3. Check generated workflows and branch-protection rulesets for drift, and require human workstation review when upgrading shared tooling pins.
4. Provide local composite-build hooks and commit-SHA staging feeds so multi-repo changes are verified end-to-end before reserving public versions.
5. Enforce `implementation`-by-default dependency scoping, automated ABI/wire compatibility checks, and ecosystem BOM alignment.
6. Isolate all signing keys, encrypted setup passphrases, and deployment credentials strictly outside AI agent sandboxes.
7. Emit an immutable **Release Attestation** recording source SHA, artifact digest, resolved lockfiles, tooling version, and verification/QA proofs for every release or deployment.
8. Keep publication and deployment explicit, idempotent operations with preserved state for recovery.

## Source references

Inspected on 2026-11-11. Links identify the studied revisions.

- [Setup and release documentation](https://github.com/lambdawalker/gradle.android-repo-setup/blob/8ec4c85a68c10e5bdb85bf01bd8bddcbfb8dfe97/README.md).
- [Gradle library plugin](https://github.com/lambdawalker/gradle.android-repo-setup/blob/8ec4c85a68c10e5bdb85bf01bd8bddcbfb8dfe97/src/main/kotlin/com/apexfission/release/AndroidRepoSetupPlugin.kt).
- [Workflow generation and drift handling](https://github.com/lambdawalker/gradle.android-repo-setup/blob/8ec4c85a68c10e5bdb85bf01bd8bddcbfb8dfe97/cli/src/workflows.rs).
- [Publication workflow template](https://github.com/lambdawalker/gradle.android-repo-setup/blob/8ec4c85a68c10e5bdb85bf01bd8bddcbfb8dfe97/cli/templates/publish-library.yml.j2).
- [Demo Common configuration](https://github.com/lambdawalker/android.apexfission.demo-common/blob/bbca41928efdb7ad7cbe71a53d2ec75095a2d047/.project/setup/config.yaml) and [plugin application](https://github.com/lambdawalker/android.apexfission.demo-common/blob/bbca41928efdb7ad7cbe71a53d2ec75095a2d047/demo-common/build.gradle.kts).
- [Permissions configuration](https://github.com/lambdawalker/android.apexfission.permissions/blob/366c04f5c28cb2268316c347005c729464e4a514/.project/setup/config.yaml) and [plugin application](https://github.com/lambdawalker/android.apexfission.permissions/blob/366c04f5c28cb2268316c347005c729464e4a514/permission/build.gradle.kts).
- [Auth publication configuration](https://github.com/lambdawalker/android.apexfission.auth/blob/6a360ca493f77a93ffe529496ef93622e38c02d8/auth/build.gradle.kts).
- [Coordinates artifact type and build](https://github.com/lambdawalker/android.apexfission.math.coordinates/blob/3f7e0c26734ebe4ed8f5644ec5df03740/README.md).
- [Application Pulumi composition](https://github.com/lambdawalker/go.attestra.aws/blob/fb63b7829308725d8c6ece6de6511efa9838e534/infra/main.go) and [Android configuration export](https://github.com/lambdawalker/go.attestra.aws/blob/fb63b7829308725d8c6ece6de6511efa9838e534/tools/deploy/internal/android/export.go).
