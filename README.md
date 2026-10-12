# markazi-updates

Official APK update channel for the **Markazi** Android app (`com.aistudio.markazi.app`).

The installed app fetches this repository's `update.json` directly over HTTPS; there is
no server or GitHub Pages involved.

## Layout

```
markazi-updates/
├── update.json              # live manifest the app reads (points at the current APK)
├── Markazi-<version>-<code>.apk   # the single current release the manifest references
├── archive/                 # every previous release, kept for rollback/audit
└── README.md
```

## How the app consumes this

- Manifest URL (fixed in the app): `https://raw.githubusercontent.com/osamasayed5/markazi-updates/main/update.json`
- `update.json` fields: `versionCode`, `versionName`, `apkUrl`, `sha256`, `sizeBytes`, `releaseNotes`.
- An update is offered only when `manifest.versionCode > installedVersionCode`
  (`UpdatePolicy.evaluate`, `NOT_NEWER` otherwise).
- The APK at `apkUrl` is verified against `sha256` and `sizeBytes` before install, and the
  package must be signed with the release certificate (`CN=Markazi Release`).

## Publishing a new release

1. Bump `versionCode`/`versionName` in the app's `app/build.gradle.kts`.
2. Build the signed release APK (`scripts/build-signed-release.ps1` in the app repo).
3. Copy the APK here as root-level `Markazi-<versionName>-<versionCode>.apk`.
4. Generate `update.json` (`scripts/prepare-update.ps1`) so `apkUrl`, `sha256`, and
   `sizeBytes` match the new APK exactly.
5. Move the previous root APK into `archive/`, then commit and push `main`.

## Rules

- **The current APK stays at the repository root** so its `apkUrl` never changes for an
  already-published manifest.
- Never modify `update.json` for an already-published version; publish a new `versionCode`.
- `archive/` is history only and is never referenced by the live manifest.
