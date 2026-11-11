# Case study: the Auth ecosystem

## Scope and status

Source study on 2026-11-11 of the Android Auth ecosystem, its shared setup tooling, and the backend composition it connects to.

The implementation and tooling rollout are in progress. This study explains the architectural direction; it is not a completed security audit, proof of correct behavior, release confirmation, or validation of a live deployment. No setup, publication, or deployment commands were executed.

The coordinates extraction motivation below is the project owner's account. Dependency declarations, tests, and composition behavior described as observed come from the referenced source.

## Android composition

The Auth repository contains a publishable `:auth` library and a local `:app` host. Its feature dependencies are maintained in separate repositories and consumed as artifacts.

| Layer or component | Observed relationship |
| --- | --- |
| Android Auth | Uses Permissions and Card Detector through version catalog dependencies. |
| Card Detector | Exposes Coordinates and YOLO dependencies through Gradle `api`. |
| Coordinates | A focused Kotlin/JVM geometry library for pixel/normalized coordinates, image spaces, and transformations. |
| Card model | Auth resolves the model release separately and copies only model assets into its library. It deliberately avoids importing the model AAR's classes and older pinned core. |
| Demo Common | Card Detector's version catalog declares the shared demo UI artifact; it serves demo infrastructure rather than the auth runtime responsibility. |

The artifact chain is visible from Coordinates through Card Detector to Auth. Each repository can establish its own implementation and release evidence while higher layers compose the resulting modules.

These statements describe declarations inspected in current source. Auth pins older dependency releases; the current Card Detector main branch and the detector artifact pinned by Auth are not assumed to be identical. No resolved dependency graph or compatibility test was run for this study.

## Backend composition

The application repository `go.attestra.aws` registers authentication, email verification, card capture, and an application-owned runtime binding.

Its `composition.go` explicitly injects contracts between those features. Its Pulumi program binds the validated resource plan to native factories from the independently maintained feature repositories.

| Owner | Observed responsibility |
| --- | --- |
| Feature repositories | Declare capabilities and requirements and provide runtime code, artifacts, and native infrastructure entry points. |
| Application repository | Selects features, binds shared resources, supplies configuration, and coordinates deployment. |
| Composer | Resolves declared contracts and prerequisites into a validated plan. |
| Pulumi adapter and factories | Register infrastructure using the application-provided factory registry and typed outputs. |
| Deployment tooling | Handles operator workflows and exports public settings consumed by Android. |

Composer's core is separate from its optional Pulumi adapter. Its documented matching checks include contract identity, major version, fields, sensitivity, and labels; full minor/patch range solving is not implemented.

The Android client consumes the backend through HTTP and configuration. It does not import Pulumi or backend runtime code. Backend composition and Android artifact composition are related parts of the system, with distinct integration mechanisms.

## Why the coordinates repository is justified

The project owner reports that coordinate logic caused persistent overlay problems. Extracting it created a bounded place to solve and verify that logic before consuming it as a library.

The current repository contains explicit image-space models and tests for transformations, parent/child spaces, normalized operations, and transformation chains. Card Detector also retains preview-space integration tests, including differing portrait and landscape aspect ratios.

This illustrates two complementary verification scopes:

- The extracted library checks the coordinate operations and their contracts.
- The consumer checks how those operations are applied to camera preview and overlay behavior.

The presence of tests is evidence of intended coverage, not a claim that tests were run or that extraction established complete correctness.

### Principle established by this example

**A module boundary can be justified by the need to isolate and verify problematic logic, even when the module is small, technically focused, and has only one consumer.**

A module can remain a library after the immediate problem is solved. Reintegration is optional and should offer a concrete benefit that outweighs its cost and the loss of an established verification boundary.

Feature and platform boundaries are starting points. Difficult algorithms, shared technical primitives, and repository automation can also be legitimate modules.

## Automation supports the repository model

Demo Common and Permissions already apply the shared setup plugin. Auth's inspected source still uses its earlier publishing setup.

The intended model is independently owned modules supported by common, versioned setup and delivery machinery. The common standard reduces repeated maintenance while module configuration preserves specific artifact and verification requirements.

See [repository automation and delivery](../repository-automation.md) for the observed responsibilities, adoption state, and proposed general rules.

## Implications and remaining questions

The example supports module boundaries based on cohesive responsibility and verification needs. It also shows that one repository can legitimately include a library plus its demo, tests, documentation, and delivery configuration.

The methodology still needs to specify:

- A module's required contract and evidence package.
- How consumer compatibility is checked against released artifacts.
- How backend outputs and client configuration versions are tracked together.
- How shared tooling upgrades are reviewed and rolled out.
- How isolated verification and whole-system auditing combine into a release decision.

## Source references

Links identify the studied revisions.

- [Auth Gradle modules](https://github.com/lambdawalker/android.apexfission.auth/blob/6a360ca493f77a93ffe529496ef93622e38c02d8/settings.gradle.kts), [dependency catalog](https://github.com/lambdawalker/android.apexfission.auth/blob/6a360ca493f77a93ffe529496ef93622e38c02d8/gradle/libs.versions.toml), and [library dependencies and model asset handling](https://github.com/lambdawalker/android.apexfission.auth/blob/6a360ca493f77a93ffe529496ef93622e38c02d8/auth/build.gradle.kts).
- [Card Detector dependency catalog](https://github.com/lambdawalker/android.apexfission.carddetector/blob/56585c3d4e00c1fe1bb94e791387679140d2d752/gradle/libs.versions.toml) and [exported library dependencies](https://github.com/lambdawalker/android.apexfission.carddetector/blob/56585c3d4e00c1fe1bb94e791387679140d2d752/carddetector/build.gradle.kts).
- [Coordinates scope and extraction origin](https://github.com/lambdawalker/android.apexfission.math.coordinates/blob/3f7e0c26734ebe4ed8f5516e0f5644ec5df03740/README.md) and [transformation-chain test](https://github.com/lambdawalker/android.apexfission.math.coordinates/blob/3f7e0c26734ebe4ed8f5516e0f5644ec5df03740/src/test/java/com/apexfission/android/carddetectionlite/domain/coordinates/TransformationChainTest.kt).
- [Card Detector preview integration tests](https://github.com/lambdawalker/android.apexfission.carddetector/blob/56585c3d4e00c1fe1bb94e791387679140d2d752/carddetector/src/test/java/com/apexfission/android/carddetector/ui/camerapreview/CreatePreviewImageSpaceChainTest.kt).
- [Backend feature registration and injection](https://github.com/lambdawalker/go.attestra.aws/blob/fb63b7829308725d8c6ece6de6511efa9838e534/composition.go) and [native Pulumi factories](https://github.com/lambdawalker/go.attestra.aws/blob/fb63b7829308725d8c6ece6de6511efa9838e534/infra/main.go).
- [Composer capabilities and limitations](https://github.com/lambdawalker/go.infra.composer/blob/ad4cfd268616a1ef94b324fe88266f845294f57a/README.md).
- [Backend Android export](https://github.com/lambdawalker/go.attestra.aws/blob/fb63b7829308725d8c6ece6de6511efa9838e534/tools/deploy/internal/android/export.go) and [Android configuration consumption](https://github.com/lambdawalker/android.apexfission.auth/blob/6a360ca493f77a93ffe529496ef93622e38c02d8/README.md).
