# Crewday Native Apps

This repository is the workspace for Crewday native applications, especially
iOS and Android.

The main Crewday web application lives in `../crewday` and already provides a
strong mobile experience. Native app work should leverage that product surface
wherever possible and add platform-specific behavior only where it improves the
app meaningfully.

## Intended Layout

- `ios/`: iOS app project and platform-specific code.
- `android/`: Android app project and platform-specific code.
- `shared/`: cross-platform contracts, generated clients, fixtures, scripts, or
  other code that is genuinely shared.
- `docs/`: architecture notes, platform decisions, and integration plans.
- `scripts/`: local automation for building, validating, generating, and
  releasing.

## Architecture Bias

Start with a thin native or hybrid shell around the existing web app, then move
specific flows native only when there is a clear reason, such as push
notifications, deep links, camera/files, location, offline behavior, platform
distribution requirements, or native performance.

Avoid duplicating web app business logic in this repository. Prefer shared APIs,
documented contracts, or integration points with `../crewday`.

