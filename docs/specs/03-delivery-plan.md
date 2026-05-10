# 03 - Delivery Plan

This plan makes scope boundaries explicit. A phase is not complete until
its acceptance criteria pass on both platforms or the skipped platform is
called out in the task.

## Phase 0 - Documentation and Decisions

Status: in progress.

Deliverables:

- Native product scope documented.
- Web app contract documented.
- Platform requirements documented.
- Architecture decision for shell baseline recorded.
- Required unresolved decisions listed below.

Acceptance criteria:

- A contributor can tell whether a requested feature belongs in
  `../crewday` or this repository.
- A contributor can tell which web routes and API endpoints the native
  shell may depend on.
- All unknowns are listed as blocked decisions rather than implied by
  prose.

## Phase 1 - Minimal Shell

Purpose: prove installable iOS and Android apps can load the existing
Crewday app and preserve web behavior.

Deliverables:

- iOS project under `ios/`.
- Android project under `android/`.
- Build configuration for development, dev/test, and production base URLs.
- Native web container loads the configured base URL.
- External-host navigation policy implemented.
- Basic native connection error and retry UI.
- No native product screens.

Acceptance criteria:

- Clean install opens the configured Crewday app.
- Passkey sign-in works on a real HTTPS host or the limitation is
  documented against the chosen web-container approach.
- `/w/<slug>/today`, `/w/<slug>/dashboard`, `/w/<slug>/chat`, and
  `/select-workspace` render through the shell.
- Session persists after app restart.
- Unknown external hosts do not load inside the Crewday web container.
- iOS and Android can be built from documented commands.

## Phase 2 - Deep Links and Association Files

Purpose: make OS links open the app without custom schemes.

Deliverables:

- iOS universal links and webcredentials entitlements.
- Android app links.
- Deployment instructions for `apple-app-site-association` and
  `assetlinks.json`.
- Cold-start and warm-start URL routing tests.

Acceptance criteria:

- An HTTPS Crewday workspace URL opens the installed app from cold start.
- The exact path, query string, and hash are preserved.
- `/w/<slug>/chat#<message_id>` opens the chat route with the fragment
  intact.
- `/w/<slug>/accept/<token>` and `/w/<slug>/guest/<token>` open in the
  app.
- Non-Crewday HTTPS links open outside the app.

## Phase 3 - Native Push Registration

Purpose: let the server deliver agent-message notifications through APNS
and FCM when deployment credentials are configured.

Native push is designed from the start but enabled later. Phase 1 must not
block on APNS or FCM credentials. Until credentials exist, the app skips
registration and the server continues to return `501 push_unavailable` for
native push-token registration.

Deliverables:

- APNS registration on iOS.
- FCM registration on Android.
- Push-token registration using `POST /api/v1/me/push-tokens`.
- Token refresh using `PUT /api/v1/me/push-tokens/{id}`.
- Sign-out cleanup using `DELETE /api/v1/me/push-tokens/{id}`.
- Handling for `501 push_unavailable` and `409 token_claimed`.

Acceptance criteria:

- Denying notification permission leaves the app usable.
- Granting permission registers a token after sign-in.
- Reinstall/sign-out does not leave the app stuck on `token_claimed`.
- A push notification opens the Crewday HTTPS deep link in the app.
- Push payload content is not treated as authoritative product data.

## Phase 4 - Native Capability Adapters

Purpose: improve OS handoff for capabilities that the web app already
uses.

Allowed adapters:

- Camera capture for task evidence and receipt scanning.
- Photo library picker for uploads.
- Document picker for attachments.
- QR/asset scan handoff if the web flow needs better native support.

Acceptance criteria:

- Every adapter starts from a user action in the web UI.
- Denied permission has a recoverable path.
- Temporary files are deleted when no longer needed.
- The web app remains the owner of uploaded-object semantics.

## Blocked Decisions

These values must be answered before implementation reaches the phase
that needs them.

| Decision | Needed by | Default until answered |
|----------|-----------|------------------------|
| Local development base URL policy | Phase 1 | Use loopback defaults from `../crewday`; passkey and link tests use dev/test |
| Apple Developer organization account and Team ID | Phase 2 | Do not publish iOS association files or enable APNS |
| Google Play organization account and signing key | Phase 2 | Do not publish `assetlinks.json` for production |
| APNS topic/key per environment | Phase 3 | Native push registration remains off |
| FCM project/sender config per environment | Phase 3 | Native push registration remains off |
| Crash-reporting provider, if any | Before beta | Native crash reports disabled |

## Change Control

Any change that adds native product UI, local domain data, a cross-
platform framework, background location, contacts, calendar access,
payments, Bluetooth, NFC, or a native auth token must update
[`00-product-scope.md`](00-product-scope.md) and add a decision record
before code lands.
