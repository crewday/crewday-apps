# 01 - Web App Contract

This document is the native repository's binding contract with
`../crewday`. It summarizes the server and web-app guarantees that the
native shell may rely on.

The authoritative source remains `../crewday`. If the exact route or
payload differs, update this document after checking the source files
listed in [`00-product-scope.md`](00-product-scope.md).

## Base URL

Every native build has exactly one configured Crewday app base URL.

Production builds must use HTTPS. Development builds may use local HTTP
only for emulator/simulator smoke tests that do not exercise passkeys,
universal links, app links, or push-notification tap-through. Those flows
must be verified over HTTPS with the real dev/test host.

Canonical hosts:

- Production app: `https://app.crew.day`
- Dev/test app: `https://dev-app.crew.day`

Local automation exceptions:

- Agent/dev host from this machine: `http://127.0.0.1:8100`
- Android emulator reaching a host dev server, when testing a local
  non-passkey build: `http://10.0.2.2:8100`

Do not hard-code `dev-app.crew.day` as an agent test target. In
`../crewday`, that public hostname is protected by remote-entry auth;
local automation uses the loopback dev app instead.

## URL Rules

The native shell must treat Crewday URLs as normal HTTPS URLs.

- Authenticated workspace routes are under `/w/<workspace_slug>/...`.
- Bare-host public routes include `/`, `/signup`, `/signup/verify`,
  `/select-workspace`, `/login`, `/recover`, `/recover/enroll`,
  `/me/email/verify`, `/healthz`, `/readyz`, and `/version`.
- Bare-host admin routes are under `/admin/...`.
- Workspace-public routes include `/w/<slug>/accept/<token>` and
  `/w/<slug>/guest/<token>`.
- Hash fragments are meaningful. Preserve fragments such as
  `/w/<slug>/chat#<message_id>`.
- Query strings are meaningful. Preserve them exactly.
- There is no custom scheme. Do not create or emit `crewday://` links.

The shell may intercept only these URL categories:

- The configured Crewday app host: open in the app web container.
- Crewday HTTPS links for the same deployment host received from the OS:
  open in the app web container.
- External hosts: open with the system browser after a user gesture,
  unless a platform policy requires a confirmation prompt.
- `mailto:`, `tel:`, and other OS-owned schemes: hand off to the OS.

The shell must reject silent navigation to unconfigured hosts inside the
web container. A compromised web page must not be able to turn the app
container into an arbitrary browser.

## Route Source

The route list is owned by `../crewday/docs/specs/14-web-frontend.md`
section "Route contract" and the implementation in
`../crewday/app/web/src/App.tsx`.

Do not duplicate the full route tree in native code. Native tests may use
route fixtures copied from `../crewday/app/web/src/routes/_surface.ts`
only when those fixtures are documented as generated or mirrored.

## Authentication

Humans authenticate through the web passkey flow. The native app does not
mint or store a separate bearer token for the user.

The shell must:

- Load `/login` or the deep-linked Crewday URL and let the web app start
  the passkey ceremony.
- Preserve the `__Host-crewday_session` cookie in the platform web
  storage.
- Treat the cookie as opaque. Native code must not read, copy, log,
  transform, or sync it.
- Use normal web navigation after login. The web app decides whether the
  user lands on `/select-workspace`, `/w/<slug>/today`,
  `/w/<slug>/dashboard`, `/w/<slug>/portfolio`, or another role home.
- Clear the platform web session only as part of explicit sign-out,
  account reset, or a debug-only data reset command.

The native app must not implement a username/password fallback. Crewday
has no passwords.

## Passkeys and Associated Domains

Passkeys are bound to the Crewday hostname. A native app may share the web
passkey experience only after the deployment publishes the required
association files:

- iOS: `apple-app-site-association` plus app entitlements containing
  `webcredentials:<host>` and `applinks:<host>`.
- Android: `assetlinks.json` plus the app's package name and signing
  certificate fingerprint.

`../crewday` currently specifies these as deployment-level configuration
that ships when the native project has a signed build. Until then, passkey
flows run inside the embedded web experience exactly as they do in a
mobile browser.

## Session and Sign-Out

Sign-out order is required:

1. If a native push-token row is registered for this install, call
   `DELETE /api/v1/me/push-tokens/{id}` while the session cookie is
   still valid.
2. Invoke the web app's normal logout route or API.
3. Clear the native shell's stored push-token row id.
4. Clear WebView website data only when the user chose a full account
   reset or when platform logout cannot be trusted to remove local state.

If step 1 fails because the session is already gone, continue logout
locally. A stale token row can expire or be disabled by server-side
lifecycle rules.

## Native Push Tokens

The native push-token surface is identity-scoped and bare-host:

```text
GET    /api/v1/me/push-tokens
POST   /api/v1/me/push-tokens
PUT    /api/v1/me/push-tokens/{id}
DELETE /api/v1/me/push-tokens/{id}
```

Request body for registration:

```json
{
  "platform": "ios",
  "token": "<apns-or-fcm-token>",
  "device_label": "iPhone 15",
  "app_version": "crewday-ios/1.0.0"
}
```

Rules:

- `platform` is exactly `ios` or `android`.
- `token` is the raw APNS or FCM device token. Never log it.
- `device_label` and `app_version` are hints only.
- `POST` may return `501 push_unavailable` until the deployment has
  native push delivery configured. The app remains fully usable.
- `GET` and `DELETE` are live even when registration is unavailable.
- Store the returned `id` locally so token refresh and sign-out can
  address the same row.
- On APNS/FCM token rotation, call `PUT /api/v1/me/push-tokens/{id}`
  with the new `token`.
- On explicit sign-out, call `DELETE /api/v1/me/push-tokens/{id}`.
- If `POST` returns `409 token_claimed`, the device token is registered
  to a different user. Show a session reset path: delete local web data,
  restart sign-in, and retry registration after the new session is
  established.

Push payloads are alerts, not data sync. The server never sends the full
message body in the push payload. Tapping a notification opens the
provided Crewday HTTPS deep link and the web app fetches current content
over HTTPS.

## Web Push Is Separate

Do not confuse native push tokens with browser Web Push subscriptions.

- Native app push: `/api/v1/me/push-tokens`, identity-scoped, APNS/FCM.
- Browser Web Push: `/w/<slug>/api/v1/messaging/notifications/push/...`,
  workspace-scoped, `PushSubscription`/VAPID.

The native app uses the native push surface only.

## Native Bridges

There is no general JavaScript bridge in v0.

Allowed bridge categories, when a concrete feature needs them:

- Open external URL.
- Report native app version and platform for diagnostics.
- Register, refresh, and delete native push tokens.
- Present native camera, photo library, file picker, or QR scanner when
  the web app requests a standard browser capability and the platform
  needs a native adapter.

Bridge calls must be tiny, named, versioned, and documented here before
use. They must not expose arbitrary filesystem, contacts, calendar,
location, clipboard, or network access to web JavaScript.

## Offline Behavior

The web app owns PWA offline behavior for worker task lists and queued
completions. Native code must not create a parallel offline task store.

If the network is unavailable before the web app loads, the shell may show
a native connection error with retry. After the web app has loaded, let
the web app and its service worker handle offline states.

## User Agent and Headers

The native shell may set a descriptive User-Agent such as:

```text
crewday-ios/1.0.0 (iOS 18.0; iPhone)
crewday-android/1.0.0 (Android 15; Pixel)
```

The server must not depend on User-Agent for auth or routing. A future
`X-Crewday-Client` header is reserved by `../crewday`; do not make native
features depend on it until the web spec does.

## Error Handling

The shell handles only shell-level failures:

- Cannot reach configured base URL.
- TLS or certificate failure.
- Navigation blocked because host is not allowed.
- Push-token registration unavailable or rejected.
- Web container process crash or reload.

All product-level errors are rendered by the web app.
