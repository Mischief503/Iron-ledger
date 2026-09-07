# Iron Ledger — Android app

Iron Ledger packaged as a real Android app (via [Capacitor](https://capacitorjs.com/),
which wraps the web app in a native WebView shell). No Play Store listing —
this is set up for sideloading, with GitHub Actions building the APK for you
on every push so you never need Android Studio installed locally.

- **App name:** Iron Ledger
- **Package ID:** `com.mischief503.ironledger`
- **Min Android version:** 5.1 (API 22) or newer

## 1. Push this to GitHub

From inside this folder:

```bash
git init -b main
git add -A
git commit -m "Iron Ledger: initial Android app"
git remote add origin https://github.com/mischief503/IronLedger.git
git push -u origin main
```

(Create the empty `IronLedger` repo on GitHub first — github.com/new — don't
initialize it with a README/license there, since this folder already has its
own git history once you run `git init`.)

## 2. Get the APK

Once it's pushed, GitHub Actions builds automatically (see
`.github/workflows/build-apk.yml`). Two ways to grab the APK:

- **Actions tab → latest run → Artifacts** → download `iron-ledger-debug-apk`
  (zipped; every run gets one, including pull requests).
- **Releases** (right sidebar of the repo) → the rolling **`latest`** release
  always has the newest `iron-ledger-latest.apk` attached, rebuilt from
  whatever is on `main`. This is the easiest link to share or bookmark on
  your phone for re-downloading updates.

## 3. Install it on your phone

Android blocks installs from outside the Play Store by default:

1. Download the `.apk` on your phone (from the Release link above, or `adb push`).
2. Tap the downloaded file. Android will prompt to allow installs from that
   source (Files app / Chrome / whichever app you used) — allow it.
3. Tap install. Since this is a debug-signed build (not signed with a Play
   Store key), Android may show an "unknown app" warning — that's normal for
   sideloaded apps, tap through it.

## Updating the app later

The web app itself lives in `www/index.html` — a single self-contained file
(same file that's published as the Claude Artifact). To ship a change:

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

## Signed release builds (optional, later)

The workflow currently ships **debug-signed** APKs, which is the normal
choice for sideloading to your own device and needs no setup. If you ever
want a proper release-signed build (e.g. to eventually publish on the Play
Store), you'd generate a keystore, add it as GitHub secrets, and add a
`signingConfig` to `android/app/build.gradle` — happy to wire that up when
you actually need it.
