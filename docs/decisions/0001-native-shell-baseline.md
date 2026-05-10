# 0001 - Native Shell Baseline

Status: accepted

## Context

The Crewday web app in `../crewday` already provides the product
experience for mobile browsers. The native apps should improve
distribution, deep links, passkeys, OS push, and selected platform
handoffs without duplicating web app workflows.

The web app explicitly publishes a native-wrapper readiness contract in
`../crewday/docs/specs/14-web-frontend.md`.

## Decision

The baseline native app is a thin platform shell around the Crewday HTTPS
web app.

The shell:

- Loads one configured Crewday deployment host.
- Uses the web passkey flow and web session cookie.
- Preserves Crewday HTTPS URLs exactly.
- Handles universal links, Android app links, and push-notification taps.
- Registers APNS/FCM push tokens through `/api/v1/me/push-tokens`.
- Adds native adapters only for OS capabilities that the web app cannot
  provide by itself.
- Uses plain platform-native code for the baseline; no Capacitor, React
  Native, Flutter, or Kotlin Multiplatform layer is part of v0.
- Uses `WKWebView` on iOS.
- Uses Android `WebView` on Android.
- Designs native push-token registration into the shell, but leaves push
  disabled until APNS and FCM credentials exist.

The shell does not:

- Reimplement Crewday product screens.
- Store Crewday business data locally.
- Create a native bearer-token auth model.
- Add a custom URL scheme.
- Create one binary per workspace.

## Fixed App Identity

- Production display name: `Crewday`
- Dev/test display name: `Crewday Dev`
- iOS production bundle identifier: `day.crew.app`
- iOS dev/test bundle identifier: `day.crew.app.dev`
- Android production application ID: `day.crew.app`
- Android dev/test application ID: `day.crew.app.dev`

Separate dev/test identifiers let the dev/test app live beside the
production app on the same phone. The identifiers are intended to be
permanent after the first store release.

## Store Ownership

The App Store and Play Store accounts should be organization-owned, not
individual-owned. The accounts do not exist yet; create them before Phase
2 association-file work or any public beta distribution.

## Remaining Platform Inputs

The container architecture is fixed. The remaining inputs are operational:
Apple Developer Team ID, Google Play signing key, APNS credentials, FCM
project configuration, and optional crash-reporting provider.

Those operational inputs are tracked in
[`../specs/03-delivery-plan.md`](../specs/03-delivery-plan.md).

## Consequences

- Most product changes continue to happen in `../crewday`.
- Native releases can stay small and platform-focused.
- Push, link association, signing, and store configuration become the
  first real native work.
- If the web app lacks a route or API needed by native, the contract is
  added to `../crewday` before native code depends on it.
