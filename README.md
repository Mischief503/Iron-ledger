# Iron Ledger — Android app

Iron Ledger packaged as a real Android app (via [Capacitor](https://capacitorjs.com/),
which wraps the web app in a native WebView shell). No Play Store listing —
this is set up for sideloading, with GitHub Actions building the APK for you
on every push so you never need Android Studio installed locally.

- **App name:** Iron Ledger
- **Package ID:** `com.mischief503.ironledger`
- **Min Android version:** 5.1 (API 22) or newer

## 1. Where it lives

This repo (`Mischief503/Iron-ledger`) is the source of truth. Every push to
`main` triggers GitHub Actions (`.github/workflows/build-apk.yml`), which
builds the APK and publishes it, so there's nothing to build or run locally.

## 2. Get the APK

The rolling **`latest`** release always has the newest build attached, and
the download link never changes:

**https://github.com/Mischief503/Iron-ledger/releases/download/latest/iron-ledger-latest.apk**

Bookmark it on your phone or send it to friends. Only builds signed with the
Iron Ledger key (see "Signing key" below) are published there. Every
workflow run, including pull requests, also keeps a zipped copy under
**Actions → the run → Artifacts → `iron-ledger-debug-apk`**.

## 3. Install it on your phone

Android blocks installs from outside the Play Store by default:

1. Open the download link above on your phone.
2. Tap the downloaded file. Android will prompt to allow installs from that
   source (Chrome / Files app / whichever app you used) — allow it.
3. Tap install. It isn't a Play Store build, so Android may show an "unknown
   app" warning — that's normal for sideloaded apps, tap through it.

Later builds install straight over the top and keep your data. The one
exception: an Iron Ledger build signed with a different key (for example the
September 2026 test build) has to be uninstalled first, and anything logged
only in that copy is removed with it.

## Updating the app later

The web app itself lives in `www/index.html` (it started as the same file
that's published as the Claude Artifact), plus the Supabase client library
bundled at `www/vendor/supabase-js-2.45.4.js` so the app never depends on a
CDN and opens fine with no signal. To ship a change:

1. Edit `www/index.html`.
2. Commit and push to `main`.
3. GitHub Actions rebuilds the APK automatically; grab the new one from the
   `latest` release once the workflow finishes (Actions tab shows progress).

Note the Claude Artifact version and this `www/index.html` are two separate
copies once you start editing either one — Claude's `db` sync capability
(cross-device history) only exists on the hosted Artifact page, so inside the
APK the app automatically falls back to on-device local storage instead
(you'll see "Saving locally in this browser only" in the History tab, same
as it does in any browser without that capability). Everything else — ramp
mode, ladder mode, the rest timer, themes, languages, the calendar — works
identically.

## Friends — profiles, friend codes, streaks & challenges

The APK has a 4th tab, **Friends**, that the web Artifact doesn't have: sign
in with email + password, get a personal 7-character friend code, add
friends by code, see their workout calendar (read-only), compare streaks,
and challenge a friend head-to-head (volume / reps / sets / streak, over
3/7/14/30 days). This runs on a dedicated Supabase project (Postgres + Auth),
since a Claude Artifact page can't make outbound calls to a third-party
backend — only the APK's WebView has real internet access, so this feature
only exists there, by design. The solo logging/history/ramp/ladder features
are unaffected either way and still work fully offline.

**How it works under the hood:**
- Supabase project: `bnlwclqhhyftekzjlkff` (region `us-west-1`). The URL and
  publishable key are embedded in `www/index.html` as plain constants — this
  is normal for Supabase's publishable/anon key, since all real access
  control happens server-side via Postgres Row Level Security (every table
  has RLS on, scoped to `auth.uid()` and friendship).
- Signing up creates a `profiles` row automatically (via a database trigger)
  with a random friend code — no server code to deploy or maintain.
- Your own logged sets are pushed to Supabase's `workout_sets` table
  whenever you're signed in (in addition to the on-device local copy), so a
  friend who's connected to you can see them and your streak stays accurate.
- **Email confirmation:** Supabase's project-level "Confirm email" setting
  is **on by default**. That means after Sign Up, a friend has to click a
  confirmation link in their email before they can sign in. If you want
  frictionless sign-up for a small friend group instead, turn it off in the
  Supabase dashboard: **Authentication → Providers → Email → uncheck
  "Confirm email"**. (I don't have a tool to flip this remotely — it's a
  one-click toggle for whoever owns the Supabase project.)
- There's currently no "unfriend" action — friend requests can be sent,
  accepted, declined, or cancelled, but an accepted friendship is permanent.
  Easy to add later if you want it.

## Building locally instead (optional)

If you'd rather build on your own machine with Android Studio:

```bash
npm install
npx cap sync android
npx cap open android   # opens the android/ folder in Android Studio
```

Then use Android Studio's **Build → Build Bundle(s) / APK(s) → Build APK(s)**,
or from the command line:

```bash
cd android
./gradlew assembleDebug
# output: android/app/build/outputs/apk/debug/app-debug.apk
```

## Signing key

Android only installs an update over an existing app if both are signed with
the same key. GitHub's build machines are wiped after every run, so without
a fixed key each build would get a brand-new random one and every update
would need an uninstall (losing on-device data). So CI signs every build with
one persistent Iron Ledger key:

- The key is a PKCS12 keystore (alias `ironledger`, valid until 2056), kept
  out of the repo in two repository secrets under **Settings → Secrets and
  variables → Actions**: `IRONLEDGER_KEYSTORE_BASE64` (the keystore,
  base64-encoded) and `IRONLEDGER_KEYSTORE_PASSWORD`.
- `android/app/build.gradle` uses it whenever `IRONLEDGER_KEYSTORE_PATH` is
  set (the workflow sets it); local builds without it fall back to Android's
  default debug key.
- If the secrets are missing, CI still builds (with a throwaway key) but
  doesn't publish to the `latest` release, so nobody installs a build that
  can't be updated later.
- Keep a backup copy of the keystore and its password. GitHub secrets can't
  be read back, and if the key is ever lost, everyone has to uninstall and
  reinstall once to move to a new one.

This is still a debug build type; publishing on the Play Store would need a
release build and Google's Play App Signing, which is a separate setup.
