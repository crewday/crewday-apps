# Agent Instructions

This repository contains the native application workspace for Crewday. The web
application lives in `../crewday` and already works well on mobile browsers. Use
that as the primary product source of truth unless a task explicitly says
otherwise.

## Product Direction

- Build iOS and Android applications that leverage the existing Crewday web app
  as much as practical.
- Avoid duplicating application flows that already work well in `../crewday`.
- Prefer native implementations only where native capabilities, distribution
  requirements, performance, offline behavior, or platform UX justify them.
- Keep iOS-specific, Android-specific, and shared code clearly separated.
- Treat the native apps as companions to the web app, not as an independent
  rewrite.

## Architecture Expectations

- Start from the smallest useful native shell before adding large frameworks or
  generated project structure.
- A WebView or hybrid shell is acceptable when it preserves product quality and
  reduces duplicated logic.
- Native features should be isolated behind small platform adapters, such as
  authentication, deep links, push notifications, camera, files, location,
  calendar, contacts, payments, or offline storage.
- Shared code belongs in `shared/` only when it is genuinely reusable by both
  platforms or supports both platforms through tooling, contracts, fixtures, or
  documentation.
- Do not copy web app business logic into this repo without first checking
  whether it can stay in `../crewday` or be exposed through an API/shared
  package.

## Repository Layout

- `ios/`: iOS application code, Xcode project files, Swift/SwiftUI/UIKit code,
  and iOS-specific documentation.
- `android/`: Android application code, Gradle project files, Kotlin/Java code,
  Jetpack Compose/XML code, and Android-specific documentation.
- `shared/`: cross-platform contracts, fixtures, scripts, generated clients, or
  reusable native-facing code.
- `docs/`: architecture notes, implementation decisions, release notes, and
  platform integration plans.
- `scripts/`: local automation for builds, validation, code generation, and
  release tasks.

## Development Rules

- Read existing files before editing and follow the established structure.
- Keep changes scoped to the requested task.
- Do not introduce a native framework, dependency manager, or app scaffold
  unless the task requires it or the user has approved that direction.
- Prefer official platform APIs and stable tooling over custom infrastructure.
- Use generated files sparingly and keep generated output out of version control
  unless it is intentionally part of the source tree.
- Keep secrets, signing material, provisioning profiles, keystores, and local
  environment files out of git.
- When modifying both platforms, verify that the behavior and naming stay
  consistent across iOS and Android.
- When adding integration with `../crewday`, document the contract and any
  assumptions about routes, authentication, cookies, deep links, or APIs.

## Verification

- Run the narrowest relevant checks for the files changed.
- For iOS work, prefer Xcode build/test commands or documented scripts once an
  iOS project exists.
- For Android work, prefer Gradle build/test commands once an Android project
  exists.
- For shared scripts or tooling, run the relevant formatter, linter, or unit
  tests when available.
- If a check cannot be run locally, report exactly what was skipped and why.

