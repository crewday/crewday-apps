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

The shell does not:

- Reimplement Crewday product screens.
- Store Crewday business data locally.
- Create a native bearer-token auth model.
- Add a custom URL scheme.
- Create one binary per workspace.

## Platform Choice Still Open

This decision does not yet choose concrete containers:

- iOS: plain `WKWebView` vs a framework wrapper around `WKWebView`.
- Android: `WebView`, Chrome Custom Tabs, or Trusted Web Activity.

Those choices are required before Phase 1 implementation in
[`../specs/03-delivery-plan.md`](../specs/03-delivery-plan.md).

## Consequences

- Most product changes continue to happen in `../crewday`.
- Native releases can stay small and platform-focused.
- Push, link association, signing, and store configuration become the
  first real native work.
- If the web app lacks a route or API needed by native, the contract is
  added to `../crewday` before native code depends on it.
