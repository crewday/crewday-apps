# Crewday Native Apps

This repository owns the iOS and Android companion applications for
Crewday. The main product remains the web application in `../crewday`.

The native apps are not a rewrite of Crewday. They are thin platform
containers around the existing mobile-first web app, plus native adapters
where the operating systems require them: app distribution, HTTPS app
links, push notifications, passkey association files, camera/file
handoff, and platform storage cleanup.

## Current Status

This repository currently contains specifications only. It intentionally
does not contain an Xcode project, Gradle project, Capacitor project, or
generated native scaffold yet.

Do not add a scaffold as a side effect of documentation, planning, or API
contract work. Add native project structure only when the task explicitly
asks for implementation and the open platform choices in
[`docs/decisions/0001-native-shell-baseline.md`](docs/decisions/0001-native-shell-baseline.md)
have been resolved.

## Source of Truth

- Product behavior: `../crewday/docs/specs/`
- Production web app: `../crewday/app/web/`
- REST contract: `../crewday/docs/specs/12-rest-api.md` and
  `../crewday/docs/api/openapi.json`
- Native wrapper contract published by the web app:
  `../crewday/docs/specs/14-web-frontend.md` section
  "Native wrapper readiness"
- Native app scope and platform rules in this repository:
  [`docs/specs/`](docs/specs/)

When this repository and `../crewday` disagree, `../crewday` wins for
product behavior and API shape. Update these native docs in the same
change so the disagreement does not persist.

## Required Layout

- `ios/`: iOS application code, Xcode project files, Swift/SwiftUI/UIKit
  code, iOS entitlements, and iOS-specific documentation.
- `android/`: Android application code, Gradle project files,
  Kotlin/Java code, Android manifest/resources, and Android-specific
  documentation.
- `shared/`: contracts, fixtures, scripts, generated clients, or
  documentation genuinely shared by both platforms.
- `docs/`: architecture notes, implementation decisions, release notes,
  and platform integration plans.
- `scripts/`: local automation for builds, validation, code generation,
  and release tasks.

These directories are created when they first have real content. Empty
placeholder app scaffolds are not useful.

## Spec Index

- [`docs/specs/00-product-scope.md`](docs/specs/00-product-scope.md):
  native product scope, non-goals, and source-of-truth rules.
- [`docs/specs/01-web-contract.md`](docs/specs/01-web-contract.md):
  exact contracts with the web app for URLs, auth, sessions, push, and
  native bridges.
- [`docs/specs/02-platform-requirements.md`](docs/specs/02-platform-requirements.md):
  iOS and Android requirements that all implementations must satisfy.
- [`docs/specs/03-delivery-plan.md`](docs/specs/03-delivery-plan.md):
  implementation phases, acceptance criteria, and blocked decisions.

## Architecture Bias

The first useful native app is a platform-native shell that loads the
Crewday HTTPS web app and preserves all web routes, auth flows, and
workspace behavior. Native code is added only for platform adapters that
the web app cannot provide by itself.

No business logic moves from `../crewday` into this repository. If native
code needs data, it uses the same documented HTTPS routes and REST
endpoints as the web app.

## Verification

For documentation-only changes, run the narrowest relevant text checks.
After app projects exist, run the documented platform checks:

- iOS: `xcodebuild` build/test commands or scripts under `scripts/`.
- Android: Gradle build/test commands or scripts under `scripts/`.
- Shared tooling: the specific formatter, linter, or unit tests for the
  changed files.

If a check cannot run locally, report the exact command that was skipped
and why.
