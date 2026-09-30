# AppFoyer drift check

A GitHub Action that tells you when a pull request makes your mobile app's privacy disclosures
outdated.

Your privacy policy, Google Play *Data safety* answers and Apple *App Privacy* details describe the
SDKs, data types and permissions your build had when they were written. When a pull request adds
Firebase Analytics, a subscription SDK or a location permission, this action lists it in the job
summary, with the file and line, and says what may now be outdated — before the change ships.

It reports. It does not edit a page, publish anything, or call any server. It needs no token, no
secret and no permission, so it also runs on pull requests from forks.

## What it reports

A pull request that adds RevenueCat to an app whose pages were written without it:

```text
"Trail Notes": 2 changes
  + In-app purchases / subscriptions (data collected)  app/build.gradle.kts:28
      → Privacy policy may not declare "In-app purchases / subscriptions"
      → Google Play Data safety and Apple App Privacy answers may be outdated
      → Terms may not cover purchases and subscriptions
  + RevenueCat (SDK)  app/build.gradle.kts:28
      → Store forms: RevenueCat is not in the app's SDK list
      → Privacy policy may not name this SDK
```

Removed SDKs and permissions are reported too: the pages may now over-declare.

## Setup

1. Add `.github/workflows/appfoyer.yml`:

   ```yaml
   name: AppFoyer drift check
   on: pull_request
   permissions:
     contents: read
   jobs:
     check:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v5
         - uses: appfoyer/check-action@v1
   ```

2. Commit an `appfoyer.json` — the record of what your pages were written from. Three ways:
   - **With the [AppFoyer Claude Code plugin](https://github.com/appfoyer/claude-plugin):** say
     *"make this app store-ready"* and it writes the file; *"add the AppFoyer drift check"* adds the
     workflow above.
   - **With nothing installed:** open a pull request. The first run finds no `appfoyer.json` and
     shows, in the job summary, the file it would write from your build. Check it against your
     published pages and commit it.
   - **Locally:** `npx @appfoyer/check init`.

From then on, every pull request is compared with `appfoyer.json`.

## Inputs

| Input | Default | |
|---|---|---|
| `path` | `.` | Folder that holds `appfoyer.json`, relative to the repository root. |
| `fail-on-drift` | `false` | Fail the job when something changed. By default the job reports and passes. |

## When it reports a change

1. Update the pages and store forms the change affects. With the plugin, say *"make this app
   store-ready"* again on the same branch: it redrafts what the change touches and rewrites
   `appfoyer.json`.
2. Edited the pages yourself instead? The job summary carries the refreshed `appfoyer.json` under
   the findings, or run `npx @appfoyer/check update`.
3. Commit `appfoyer.json` in the same pull request. The next run shows no changes.

## Supported projects

Native Android and iOS, Flutter, React Native (bare and Expo) and Kotlin Multiplatform. Product
flavors with their own application id can each have their own entry in `appfoyer.json`
(`sourceSets`), and an app inside a monorepo its own folder (`root`).

## What it reads

Build files only: Gradle files and version catalogs, `AndroidManifest.xml`, `Podfile` and
`Podfile.lock`, `Package.swift`, `project.pbxproj`, `Info.plist`, `.entitlements`, `pubspec.yaml`,
`package.json` and `app.json`. It never opens source code or `.env` files and sends nothing
anywhere.

## Limitations

- A dependency listed in a build file is not proof the app uses it. Each finding says what *may* be
  outdated, not that anything is wrong. This is not legal advice.
- Dependencies pulled in by other libraries are not seen, except through `Podfile.lock`.
- `app.config.js` / `app.config.ts` (Expo) are code and are not read.
- A Gradle dependency written over several lines is missed.

## Not on GitHub?

The same check is on npm as [`@appfoyer/check`](https://www.npmjs.com/package/@appfoyer/check) and
runs on any CI with Node 20 or later: `npx -y @appfoyer/check --fail-on-drift`.

More: [Keeping your privacy policy in sync with your build](https://appfoyer.com/guides/keep-privacy-policy-in-sync-with-your-build).

## License

MIT. `dist/index.mjs` bundles third-party code; see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
