# 00 - Product Scope

## Purpose

The Crewday native apps exist to make the existing Crewday web product
installable, launchable, deep-linkable, and reachable through OS-level
notifications on iOS and Android.

They do not define a second Crewday product. The main app in `../crewday`
continues to own workflows, copy, visual design, permissions, REST
schemas, offline web behavior, and the route tree.

## Product Source of Truth

Use these files in `../crewday` before changing native behavior:

- Product overview and personas:
  `../crewday/docs/specs/00-overview.md`
- Web frontend and native-wrapper readiness:
  `../crewday/docs/specs/14-web-frontend.md`
- Authentication and session rules:
  `../crewday/docs/specs/03-auth-and-tokens.md`
- Messaging and push delivery rules:
  `../crewday/docs/specs/10-messaging-notifications.md`
- REST API route catalog:
  `../crewday/docs/specs/12-rest-api.md`
- OpenAPI document:
  `../crewday/docs/api/openapi.json`
- Production SPA implementation:
  `../crewday/app/web/`

If a native feature requires the web app to expose a new route, API,
cookie behavior, or associated-domain file, document the contract in
`../crewday` first or in the same change. Native code must not depend on
an undocumented server behavior.

## In Scope

The native apps may own:

- App launch, splash/loading state, and recovery UI when the web app
  cannot be reached.
- A secure platform web container for the Crewday HTTPS app.
- HTTPS app links and universal links for Crewday routes.
- Associated-domain and Digital Asset Links requirements for passkeys and
  app links.
- Native push-token registration with
  `POST /api/v1/me/push-tokens`.
- Push-token refresh and deletion on OS token rotation, sign-out, and app
  uninstall signals where the OS provides them.
- Camera, photo library, document picker, and QR-scan handoff where the
  embedded web app requests those capabilities.
- Local platform settings needed by the shell, such as selected
  deployment URL in development builds.
- Platform crash logs and native telemetry for shell failures, excluding
  web app business events already tracked by `../crewday`.

## Out of Scope

The native apps must not own:

- Manager, worker, client, guest, admin, or agent workflows already
  implemented in the web app.
- A second task list, schedule, chat, payroll, inventory, property,
  employee, or settings implementation.
- Local replicas of Crewday domain models except tiny DTOs required by a
  native adapter.
- A native login screen that bypasses the web passkey flow.
- A native bearer-token session model.
- Per-workspace app binaries, icons, bundle IDs, package names, or store
  listings.
- A custom URL scheme such as `crewday://`.
- Cross-deployment account switching unless a later decision explicitly
  allows it.

## App Model

One native app binary targets one Crewday deployment host. That app covers
every workspace the signed-in user belongs to, using the same workspace
switcher as the web SPA.

The native app does not install one app per workspace. Per-workspace PWA
installs remain a browser-only feature owned by `../crewday`.

## Initial Implementation Strategy

The required v0 implementation is a thin native shell:

1. Load the configured Crewday HTTPS base URL.
2. Let the web app run its existing passkey, workspace, and role-routing
   flows.
3. Preserve every HTTPS URL, query string, and hash fragment.
4. Add native adapters only after the web route works in a normal mobile
   browser.

Any proposal to use Capacitor, React Native, Flutter, Kotlin
Multiplatform, a generated OpenAPI client, or a shared native UI layer is
a new architecture decision. Do not introduce those tools without a
decision record.

## Blocked Decisions

This repository has not yet chosen:

- iOS bundle identifier.
- Android application ID.
- Production deployment host for the first shipped app.
- App Store / Play Store distribution account.
- Whether Android v0 is a `WebView`, Custom Tabs, or Trusted Web
  Activity shell.
- Whether iOS v0 is pure `WKWebView` or a larger framework wrapper.
- FCM sender/project and APNS topic/team identifiers.

These are blocked decisions, not implementation gaps. Code must not guess
them. Use [`03-delivery-plan.md`](03-delivery-plan.md) to track when a
decision is required.
