# 02 - Platform Requirements

These requirements apply to every iOS and Android implementation in this
repository.

## Shared Requirements

The native shell must:

- Load only the configured Crewday app base URL in its primary web
  container.
- Preserve route path, query string, and hash through cold start, warm
  resume, push taps, universal links, and Android app links.
- Use the web app for login, workspace selection, role routing, and all
  product workflows.
- Persist the web session using platform web storage.
- Avoid storing Crewday domain data in native storage except:
  push-token row id, selected dev base URL, app diagnostics, and
  platform permission state.
- Keep native logs free of cookies, APNS/FCM tokens, magic-link tokens,
  invite tokens, guest tokens, uploaded file names, chat message bodies,
  and property access details.
- Treat external links as external. Do not load untrusted hosts inside
  the Crewday web container.
- Use system accessibility defaults: dynamic text where supported,
  screen-reader labels for native controls, and no native gesture that
  blocks web accessibility.
- Respect OS safe areas and notches without overlaying web controls.
- Continue to function when native push is unavailable.

## iOS Requirements

The iOS app must:

- Use a stable bundle identifier chosen before the first Xcode project is
  committed.
- Load Crewday in `WKWebView` or a documented wrapper around
  `WKWebView`.
- Use an ephemeral `WKWebView` only for debug reset flows. The normal app
  must use persistent website data so the Crewday session survives app
  restarts.
- Use `ASWebAuthenticationSession` only if a later auth decision requires
  a system-browser ceremony. The default is web passkey flow inside the
  Crewday web experience.
- Declare universal link and webcredentials entitlements only after the
  deployment publishes the matching `apple-app-site-association` file.
- Request notification permission only when the product has a clear
  reason to register native push, not at first launch by default.
- Register APNS only after the user is signed in or at the point where
  the platform requires early registration; if early registration is
  unavoidable, do not send the token to Crewday until after sign-in.
- Delete the registered `/api/v1/me/push-tokens/{id}` row on explicit
  sign-out when possible.
- Support camera, photo library, and document picker requests only in
  response to user-initiated web actions.
- Avoid private Apple APIs and avoid inspecting `HttpOnly` cookies.

## Android Requirements

The Android app must:

- Use a stable application ID chosen before the first Gradle project is
  committed.
- Use Kotlin for native source unless a later decision chooses a
  different language.
- Choose exactly one web-container approach before implementation:
  Android `WebView`, Chrome Custom Tabs, or Trusted Web Activity.
  The choice is recorded in a decision file before code lands.
- Persist the web session across app restarts unless the user signs out
  or chooses debug data reset.
- Publish Android App Links only after the deployment publishes matching
  `assetlinks.json` with the package name and signing certificate
  fingerprint.
- Request notification permission on Android versions that require it
  only when enabling native push for a signed-in user.
- Register FCM only after sign-in or when platform startup requires it;
  if obtained early, do not send the token to Crewday until after
  sign-in.
- Delete the registered `/api/v1/me/push-tokens/{id}` row on explicit
  sign-out when possible.
- Support camera, photo picker, document picker, and QR handoff only in
  response to user-initiated web actions.
- Disable arbitrary file access from web content unless a specific file
  picker flow needs a scoped URI grant.

## Permissions

No permission is requested at first launch by default.

Permission prompts are tied to user action:

| Permission | Prompt trigger | Native behavior |
|------------|----------------|-----------------|
| Notifications | User enables app notifications or a signed-in push registration flow starts | Register APNS/FCM, then call `/api/v1/me/push-tokens` |
| Camera | User starts receipt scan, task evidence capture, QR scan, or asset scan | Open camera through the web-requested flow or native adapter |
| Photos | User chooses an existing image for task evidence, receipt, avatar, or document | Open picker with scoped access |
| Files | User attaches a document | Open document picker with scoped access |
| Location | Not used in v0 | Do not request |
| Contacts | Not used in v0 | Do not request |
| Calendar | Not used in v0 | Do not request |

Adding location, contacts, calendar, payments, Bluetooth, NFC, or
background execution requires a spec update before code.

## Navigation and Link Handling

Cold-start inputs:

- App icon launch: open the configured base URL.
- Universal link / Android app link: open the exact Crewday HTTPS URL.
- Push notification tap: open the deep link in the push payload after
  validating that the host is the configured Crewday host.
- External share/open-with action: unsupported in v0 unless a later spec
  adds a concrete flow.

Warm navigation:

- Same deployment host: stay inside the Crewday web container.
- Other Crewday deployment host: open externally unless the build is a
  debug build with explicit multi-host switching enabled.
- Non-Crewday host: open externally.
- Invalid, non-HTTPS production URL: block and show a native error.

## Storage

Allowed native storage:

- Configured base URL for debug builds.
- Last successful base URL reachability timestamp.
- Native push-token row id returned by Crewday.
- Native app version/build and diagnostic flags.
- Crash reports governed by the platform crash-reporting policy.

Disallowed native storage:

- Session cookies.
- API tokens, delegated tokens, or personal access tokens.
- Passkey material.
- Chat messages.
- Task, payroll, employee, client, property, guest, inventory, or asset
  records.
- Uploaded file bytes outside the temporary files required by an OS
  picker.

## Build Configuration

Each platform must have explicit build flavors or schemes for:

- Development: developer-controlled base URL, debug logging allowed, no
  production signing.
- Staging: fixed staging HTTPS base URL, release-like signing, native
  push may be enabled.
- Production: fixed production HTTPS base URL, store signing, native push
  enabled only after credentials are configured.

The base URL must be visible in build configuration. It must not be
hidden in source code literals scattered through the app.

## Verification Before Release

Before any native release candidate:

- Open the app from a clean install and sign in with a passkey.
- Open a manager workspace route and a worker workspace route at phone
  width.
- Follow a `/w/<slug>/chat#<message_id>` link from cold start.
- Follow `/w/<slug>/accept/<token>` and `/w/<slug>/guest/<token>` test
  links from cold start.
- Sign out and verify the local push-token row id is cleared.
- If push is enabled, verify APNS/FCM token registration, token refresh,
  notification tap-through, and sign-out deletion.
- Deny each requested permission and verify the web app remains usable.
- Kill and relaunch the app, then verify the session persists.
- Clear local data and verify the app returns to signed-out state.
