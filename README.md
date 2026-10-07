# LedgerReady iPhone pilot

This is source code for an Expo / React Native iPhone app, not an installable IPA and not an App Store release. It is configured for:

- Expo owner: holler-honey-media
- Project slug: ledger-ready
- EAS project ID: 164eae03-6d06-4930-88fe-0b5c26ba91de
- Proposed iOS bundle ID: com.hollerhoneymedia.ledgerready (verify availability before signing)
- Existing workspace: https://ledger-ready.chasity-west16.chatgpt.site

## What this build does

The existing LedgerReady workspace runs in an embedded WebView, preserving income, expenses, receipt OCR, documents and reports. Native iPhone controls record trips using Expo Location and a top-level background task. It requests foreground and background location permission only when a user starts a trip. Stop ends location updates and queues a record for the existing mileage API. A captured trip with missing GPS segments requires mileage confirmation. Pending trips retry explicitly or when the signed-in workspace reconnects. A stable trip ID prevents duplicate server records.

The trip queue is device-local SQLite with SQLCipher enabled in the native build; its random database key is stored in iOS Keychain. App reinstall/device loss can lose unsynced trips. Only the matching signed-in workspace can receive a queued record. Other bookkeeping records remain on the existing server. Native sharing handles report ZIPs and attached files up to 25 MB; temporary shared files are deleted after sharing. Larger files should be exported in Safari during the pilot.

This is a hybrid pilot, not a full native rewrite. The current server is owner-private and uses ChatGPT sign-in. No general customer authentication, billing, or app-owned customer identity has been added. Public launch is still pending the work below.

## Build connection still needed

A project UUID configures the target; it grants no account access. No build has been uploaded to Expo and no Apple submission has been made. Do not send passwords or access tokens in chat. Keep any EAS / Apple credentials in the build provider's secure credential store.

A connected GitHub repository is a useful phone-friendly path: keep this source in a private repository, connect it to the existing Expo project, then start EAS builds using its supported GitHub integration. If executing from a development environment, Node 22.13+ is recommended (Node 24 was used for these checks):

    npm ci
    npx eas-cli@latest login
    npx eas-cli@latest whoami
    npx eas-cli@latest device:create
    npx eas-cli@latest build --platform ios --profile development

An Apple Developer Program membership and iOS signing credentials are required to install the development build on a physical iPhone. Select/register the correct iPhone. Allow EAS to manage signing credentials if desired. Background location and SQLCipher must be tested in an installed development build, not Expo Go. EAS performs the native macOS build in the cloud.

Once installed, the app uses the live private workspace; verify that sign-in works inside the WebView before entering actual business records. If the identity provider blocks embedded sign-in, implement an approved system-browser authentication flow and app-owned server sessions before continuing. Do not weaken OAuth restrictions or embed bypass tokens.

The development profile includes a development client. Use `npm start` from a reachable development environment to serve JavaScript. For a standalone internal pilot after sign-in is validated, use the `preview` profile, which includes the JavaScript bundle:

    npx eas-cli@latest build --platform ios --profile preview

## Checks already run

- TypeScript check passed.
- Eight trip distance/state tests passed (stationary GPS noise, inaccurate/duplicate fixes, plausible travel, GPS gaps, impossible jumps, late callbacks and zero/stale mileage).
- Expo iOS JavaScript bundle export passed.
- Expo prebuild generated the iOS project successfully.
- Generated iOS configuration contains location background mode, purpose strings, and SQLCipher.
- Matching website identity/sync bridge compiled and was prepared for publication.

These checks do not prove the app installs or works on an actual iPhone. Native compilation, device permissions, WebView login, screen-lock tracking, OCR and file sharing have not yet been tested on a device.

## Required physical-device acceptance checks

1. Sign in and confirm only the correct workspace records appear. Sign out and change accounts; unsaved trips must never sync into the wrong account.
2. Start a business trip, grant foreground then Always location, drive a known route with the screen locked and with another app open. Compare against the odometer.
3. Stop while parked; verify one mileage entry, matching purpose/vehicle and trip times. Retry a save after a network timeout; verify no duplicate.
4. Disable location temporarily, force-close/reopen and test denied permissions. Missing distance must prompt review rather than silently fill a straight-line route.
5. Stop while offline; reconnect and verify the queued trip saves once. Restart the app before reconnecting to verify local persistence.
6. Photograph a receipt, review OCR, save it and check the report. Download a receipt and export a tax package via native sharing.
7. Test larger text settings, safe-area layout, keyboard behavior and VoiceOver on actual iPhones.
8. Verify SQLCipher reads while the screen is locked after first unlock, without capturing coordinates before permission.

## Before a public release

- Add app-owned customer sign-in, proper authorization and in-app account/data deletion. Evaluate Apple's login-service requirements for the actual authentication implementation.
- Review document and financial-data security, retention, backups, logging, data export, and privacy consent. Native background GPS battery use needs device measurements.
- Add a truthful privacy policy and support page, and complete privacy disclosures for precise location, financial records, user content and identity.
- Confirm individual-developer eligibility under Apple's sensitive-information guideline; do not assume approval.
- Complete encryption/export-compliance classification for SQLCipher. No exemption answer has been preselected in this configuration.
- Provide an icon, screenshots, real metadata, reviewer access and instructions. Verify ownership and signing IDs.
- Complete any applicable payment/subscription rules before charging users.
- Upload a signed production build only after acceptance checks pass. EAS Submit uploads to App Store Connect; it does not itself complete Apple review or approval.

## Source structure

- `App.tsx`: native shell, recorder, review, sync and sharing
- `src/tracking.ts`: background task and permission lifecycle
- `src/store.ts`: encrypted local trip queue
- `src/trip-math.ts`: GPS distance filtering and record payload
- `src/bridge.ts`: same-origin WebView protocol
- `eas.json`, `app.config.ts`: cloud build and iPhone capabilities
- `tests/trip-math.test.ts`: distance/state checks

Do not commit generated `ios/`, `node_modules/`, `.expo/`, secrets or builds. EAS regenerates native files from the checked-in configuration.
