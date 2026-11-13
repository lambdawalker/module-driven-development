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

### Transitive governance lessons illustrated here

Two observations in the Android chain directly motivate the transitive dependency rules in [Repository automation](../repository-automation.md#dependency-and-transitive-version-governance):

- **Manual AAR asset extraction as a boundary smell:** Auth's build logic unpacks model files from the Card model release artifact to avoid pulling in the model AAR's classes and older transitive core dependency. Packaging ML weights and metadata manifests as code-free asset artifacts eliminates the need for consumers to strip classes manually.
- **Explicit `api` versus `implementation` scoping:** Card Detector exports `Coordinates` and `YOLO` via Gradle `api`. Under the proposed Gradle reference policy, `api` exposure should be reserved strictly for types intentionally surfaced in Card Detector's public contract, internalizing underlying inference details behind `implementation` and guarding public signatures with automated ABI checks.

## Backend composition

The application repository `go.attestra.aws` registers authentication, email verification, card capture, and an application-owned runtime binding.

Its `composition.go` explicitly injects contracts between those features. Its Pulumi program binds the validated resource plan to native factories from the independently maintained feature repositories.

| Owner | Observed responsibility |
| --- | --- |
| Feature repositories | Declare capabilities and requirements and provide runtime code, artifacts, and native infrastructure entry points. |
| Application repository (`go.attestra.aws`) | Acts as an **Application Composition Host** (see [Principles](../principles.md#distinguish-leaf-modules-from-composition-hosts)): selects features, binds shared resources, enforces system configuration, and coordinates deployment. |
| Composer | Resolves declared contracts and prerequisites into a validated plan. |
| Pulumi adapter and factories | Register infrastructure using the application-provided factory registry and typed outputs. |
| Deployment tooling | Handles operator workflows and exports public settings consumed by Android. |

Composer's core is separate from its optional Pulumi adapter. Its documented matching checks include contract identity, major version, fields, sensitivity, and labels; full minor/patch range solving is not implemented.

The Android client consumes the backend through HTTP and exported configuration. It does not import Pulumi or backend runtime code. Because shipped Android APKs cannot be reverted once installed on user devices, this HTTP/configuration boundary requires explicit N-back wire compatibility and server-side kill switches during rollout.

## Why the coordinates repository is justified

The project owner reports that coordinate logic caused persistent overlay problems. Extracting it created a bounded place to solve and verify that logic before consuming it as a library.

The current repository contains explicit image-space models and tests for transformations, parent/child spaces, normalized operations, and transformation chains. Card Detector also retains preview-space integration tests, including differing portrait and landscape aspect ratios.

This illustrates two complementary verification scopes:

- The extracted library (pure-compute behavior; risk depends on its consumers) checks coordinate invariants and mathematical contracts without camera or UI noise.
- The consumer (feature integration behavior; risk requires project assessment) checks how those operations apply to camera preview spaces and overlay rendering.

The presence of tests is evidence of intended coverage, not a claim that tests were run or that extraction established complete correctness. Pairing `Coordinates` with context-isolated invariant testing further protects against correlated AI assumptions.

### Principle established by this example

**A module boundary can be justified by the need to isolate and verify problematic logic, even when the module is small, technically focused, and has only one consumer.**

A module can remain a library after the immediate problem is solved. Reintegration is optional and should offer a concrete benefit that outweighs its cost and the loss of an established verification boundary.

Feature and platform boundaries are starting points. Difficult algorithms, shared technical primitives, and repository automation can also be legitimate modules.

## Automation supports the repository model

Demo Common and Permissions already apply the shared setup plugin. Auth's inspected source still uses its earlier publishing setup.

The intended model is independently owned modules supported by common, versioned setup and delivery machinery. Extending that machinery with local composite builds and commit-SHA CI staging feeds allows coordinated updates across `Coordinates` → `Card Detector` → `Auth` to be verified before cutting public releases; see [Repository automation and delivery](../repository-automation.md).

## Implications and remaining questions

The example supports module boundaries based on cohesive responsibility and verification needs, clarifies the distinction between **leaf module repositories** and **application composition hosts**, and shows that one leaf repository can legitimately include a library plus its demo host, tests, documentation, and delivery configuration.

Proposed ecosystem improvements to assess in separate implementation work include:

- Adopting standard `Module Specification` (`MODULE.md`), `Change Agreement`, and `Release Attestation` files across these repositories.
- Migrating ML model artifacts to code-free bundles and auditing `api` vs. `implementation` exposure with binary-compatibility checks.
- Adding commit-SHA staging feeds so consumer integration branches verify upstream candidates prior to public tag publication.
- Versioning backend exported Android configurations alongside N-back HTTP contract checks.
- Completing the `gradle.android-repo-setup` migration across remaining Android modules with Signing-Key isolation and enforced branch rulesets.

## Source references

Links identify the studied revisions.

- [Auth Gradle modules](https://github.com/lambdawalker/android.apexfission.auth/blob/6a360ca493f77a93ffe529496ef93622e38c02d8/settings.gradle.kts), [dependency catalog](https://github.com/lambdawalker/android.apexfission.auth/blob/6a360ca493f77a93ffe529496ef93622e38c02d8/gradle/libs.versions.toml), and [library dependencies and model asset handling](https://github.com/lambdawalker/android.apexfission.auth/blob/6a360ca493f77a93ffe529496ef93622e38c02d8/auth/build.gradle.kts).
- [Card Detector dependency catalog](https://github.com/lambdawalker/android.apexfission.carddetector/blob/56585c3d4e00c1fe1bb94e791387679140d2d752/gradle/libs.versions.toml) and [exported library dependencies](https://github.com/lambdawalker/android.apexfission.carddetector/blob/56585c3d4e00c1fe1bb94e791387679140d2d752/carddetector/build.gradle.kts).
- [Coordinates scope and extraction origin](https://github.com/lambdawalker/android.apexfission.math.coordinates/blob/3f7e0c26734ebe4ed8f5516e0f5644ec5df03740/README.md) and [transformation-chain test](https://github.com/lambdawalker/android.apexfission.math.coordinates/blob/3f7e0c26734ebe4ed8f5516e0f5644ec5df03740/src/test/java/com/apexfission/android/carddetectionlite/domain/coordinates/TransformationChainTest.kt).
- [Card Detector preview integration tests](https://github.com/lambdawalker/android.apexfission.carddetector/blob/56585c3d4e00c1fe1bb94e791387679140d2d752/carddetector/src/test/java/com/apexfission/android/carddetector/ui/camerapreview/CreatePreviewImageSpaceChainTest.kt).
- [Backend feature registration and injection](https://github.com/lambdawalker/go.attestra.aws/blob/fb63b7829308725d8c6ece6de6511efa9838e534/composition.go) and [native Pulumi factories](https://github.com/lambdawalker/go.attestra.aws/blob/fb63b7829308725d8c6ece6de6511efa9838e534/infra/main.go).
- [Composer capabilities and limitations](https://github.com/lambdawalker/go.infra.composer/blob/ad4cfd268616a1ef94b324fe88266f845294f57a/README.md).
- [Backend Android export](https://github.com/lambdawalker/go.attestra.aws/blob/fb63b7829308725d8c6ece6de6511efa9838e534/tools/deploy/internal/android/export.go) and [Android configuration consumption](https://github.com/lambdawalker/android.apexfission.auth/blob/6a360ca493f77a93ffe529496ef93622e38c02d8/README.md).
