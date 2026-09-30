# AppFoyer drift check

A GitHub Action that tells you when a pull request makes your app's privacy disclosures outdated.

Your privacy policy, Google Play *Data safety* answers and Apple *App Privacy* details were written
from what your app's build contained at one moment. When a pull request adds Firebase Analytics, a
location permission or an ad SDK, this action lists it in the job summary, with the file and line,
and says what may now be outdated.

It reports. It does not edit a page, publish anything, or call any server.

## Setup

1. Add `.github/workflows/appfoyer.yml`:

   ```yaml
   name: AppFoyer drift check
   on: pull_request
   jobs:
     check:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - uses: appfoyer/check-action@v1
   ```

2. Open a pull request. The first run finds no `appfoyer.json` and shows, in the job summary, the file
   it would write from your build. Check it against your published pages and commit it at the
   repository root.

From then on, every pull request is compared with `appfoyer.json`.

## Inputs

| Input | Default | |
|---|---|---|
| `path` | `.` | Folder that holds `appfoyer.json`, relative to the repository root. |
| `fail-on-drift` | `false` | Fail the job when something changed. By default the job reports and passes. |

## When it reports a change

Update the pages first: run "make this app store-ready" with the plugin, or edit the app in the
AppFoyer dashboard. Then refresh `appfoyer.json` in the same pull request.

## What it reads

Build files only: Gradle files and version catalogs, `AndroidManifest.xml`, `Podfile` and
`Podfile.lock`, `Package.swift`, `project.pbxproj`, `Info.plist`, `.entitlements`, `pubspec.yaml`,
`package.json` and `app.json`. It never opens source code or `.env` files, needs no token and no
permission, and runs on pull requests from forks.

## Limitations

- A dependency listed in a build file is not proof the app uses it. Each finding says what *may* be
  outdated, not that anything is wrong. This is not legal advice.
- Dependencies pulled in by other libraries are not seen, except through `Podfile.lock`.
- `app.config.js` / `app.config.ts` (Expo) are code and are not read.
- A Gradle dependency written over several lines is missed.

## License

MIT. `dist/index.mjs` bundles third-party code; see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
