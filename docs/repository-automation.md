# Repository automation and delivery

## Direction established in discussion

Independent repositories need shared automation for recurring setup and consistent delivery. The Android standard is being implemented through [gradle.android-repo-setup](https://github.com/lambdawalker/gradle.android-repo-setup); adoption is in progress.

A shared tool is itself a module with a version, consumers, maintenance responsibilities, and audit scope.

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

A published Android library and a deployed backend are different deliverables. Both need traceable versions and verification, but they use different tools and have different evidence requirements.

## Proposed methodology rules

The following are proposals for extending the observed approach into a general standard.

1. Maintain shared setup and delivery behavior in versioned tooling instead of independently reproducing it in each repository.
2. Keep repository-specific identity, artifact type, commands, and verification expectations in reviewable configuration.
3. Check generated files for drift and review changes when upgrading tooling.
4. Verify the artifact a consumer will receive, including its dependencies and public API, as well as the module source.
5. Record source, configuration, tooling versions, destination, and outcome for each release or deployment.
6. Keep publication and deployment explicit operations with recoverable state.
7. Validate consuming integrations when changing a module, tooling, or environment configuration.
8. Audit shared automation with the same attention as product modules: it can hold credentials, execute code, publish artifacts, and affect many consumers.

Required configuration fields, supported artifact profiles, release gates, and deployment evidence formats remain open.

## Source references

Inspected on 2026-11-11. Links identify the studied revisions.

- [Setup and release documentation](https://github.com/lambdawalker/gradle.android-repo-setup/blob/8ec4c85a68c10e5bdb85bf01bd8bddcbfb8dfe97/README.md).
- [Gradle library plugin](https://github.com/lambdawalker/gradle.android-repo-setup/blob/8ec4c85a68c10e5bdb85bf01bd8bddcbfb8dfe97/src/main/kotlin/com/apexfission/release/AndroidRepoSetupPlugin.kt).
- [Workflow generation and drift handling](https://github.com/lambdawalker/gradle.android-repo-setup/blob/8ec4c85a68c10e5bdb85bf01bd8bddcbfb8dfe97/cli/src/workflows.rs).
- [Publication workflow template](https://github.com/lambdawalker/gradle.android-repo-setup/blob/8ec4c85a68c10e5bdb85bf01bd8bddcbfb8dfe97/cli/templates/publish-library.yml.j2).
- [Demo Common configuration](https://github.com/lambdawalker/android.apexfission.demo-common/blob/bbca41928efdb7ad7cbe71a53d2ec75095a2d047/.project/setup/config.yaml) and [plugin application](https://github.com/lambdawalker/android.apexfission.demo-common/blob/bbca41928efdb7ad7cbe71a53d2ec75095a2d047/demo-common/build.gradle.kts).
- [Permissions configuration](https://github.com/lambdawalker/android.apexfission.permissions/blob/366c04f5c28cb2268316c347005c729464e4a514/.project/setup/config.yaml) and [plugin application](https://github.com/lambdawalker/android.apexfission.permissions/blob/366c04f5c28cb2268316c347005c729464e4a514/permission/build.gradle.kts).
- [Auth publication configuration](https://github.com/lambdawalker/android.apexfission.auth/blob/6a360ca493f77a93ffe529496ef93622e38c02d8/auth/build.gradle.kts).
- [Coordinates artifact type and build](https://github.com/lambdawalker/android.apexfission.math.coordinates/blob/3f7e0c26734ebe4ed8f5516e0f5644ec5df03740/README.md).
- [Application Pulumi composition](https://github.com/lambdawalker/go.attestra.aws/blob/fb63b7829308725d8c6ece6de6511efa9838e534/infra/main.go) and [Android configuration export](https://github.com/lambdawalker/go.attestra.aws/blob/fb63b7829308725d8c6ece6de6511efa9838e534/tools/deploy/internal/android/export.go).
