# Buzz — unofficial Android builds

Automated, unofficial CI builds of the [block/buzz](https://github.com/block/buzz) mobile app
(Apache-2.0), for people who want Buzz on Android via [Obtainium](https://github.com/ImranR98/Obtainium)
— upstream currently publishes no Android APKs anywhere.

**Not affiliated with or endorsed by Block.** Built verbatim from upstream source at their
`mobile-v*` release-candidate tags, using their Hermit-pinned toolchain, in public GitHub Actions
(auditable: see [`.github/workflows/build.yml`](.github/workflows/build.yml)).

## Install with Obtainium

Tap on the device with Obtainium installed:

**[`obtainium://add/https://github.com/fredabood/buzz`](obtainium://add/https://github.com/fredabood/buzz)**

or add `https://github.com/fredabood/buzz` manually. Default settings work — every release has
exactly one universal APK, marked non-prerelease. New upstream mobile tags are detected within
6 hours and released automatically.

## Verify what you install

Every APK is signed with the same personal certificate:

```
SHA-256: C0:20:5E:95:FC:AA:56:DC:21:8B:8A:82:9B:6A:C5:3D:28:BD:C5:E4:55:89:B1:1D:22:18:81:A0:29:99:30:D7
```

Check with `apksigner verify --print-certs buzz-mobile-*.apk`. CI refuses to publish if a build's
certificate deviates from this fingerprint.

## Caveats

- **Signature conflict with any future official build.** These APKs keep upstream's
  `applicationId` (`xyz.block.buzz.mobile`) but use a different signing key than anything Block
  might ship later. Android forbids cross-signature updates: to switch to an official APK you
  must uninstall this one first (chat data lives on your relays; your identity is your Nostr key,
  so loss is minimal — export/back up your key first regardless).
- The compile-time default relay URL is baked to `wss://buzz.dirtydata.studio` (the maintainer's
  self-hosted community, tailnet-only) instead of upstream's `http://localhost:3000` fallback.
  This only affects the out-of-box default — communities are added/switched in-app as usual.
- Versions mirror upstream `mobile-vX.Y.Z-rc.N` tags. Upstream cuts release candidates roughly
  weekly; whatever they tag is what gets built, unmodified.
- If upstream starts publishing official Android builds
  ([block/buzz#2845](https://github.com/block/buzz/issues/2845)), prefer those — this repo exists
  only to fill that gap and will be archived when it closes.

## How it works

`build.yml` runs every 6 hours (and on manual dispatch):

1. Lists upstream `mobile-v*` tags (upstream has tags but no mobile releases), picks the highest
   by rc-aware semver, and exits early if this repo already has that release.
2. Checks out `block/buzz` at the tag; activates upstream's own Hermit toolchain (Flutter, JDK).
3. Builds `flutter build apk --release` with `--build-name`/`--build-number` derived from the tag
   (upstream's pubspec is intentionally `0.0.0+1`; version injection is the release pipeline's job).
4. Signs via upstream's `BUZZ_ANDROID_UPLOAD_*` environment contract with a keystore held only in
   GitHub Actions secrets; verifies the certificate fingerprint before publishing.
5. Publishes the APK as a GitHub Release named after the upstream tag.
