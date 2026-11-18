# Releasing Parallax

Pushing a `v*` tag runs `.github/workflows/release.yml` on a GitHub-hosted macOS runner. It:

1. runs the client tests (`client/scripts/test.sh`),
2. builds `Parallax.app` in release mode with the bundled `parallax-server` (`client/scripts/build-app.sh release`),
3. zips the app with `ditto`, writes a SHA-256 checksum, and attaches both to the GitHub Release for the tag (creating a draft release if none exists).

The client is a Swift package, not an Xcode project, so there is no `xcodebuild archive` step. `build-app.sh` assembles the `.app` bundle itself.

## Signing and notarization (future work)

Release builds are ad hoc signed only. They are not signed with a Developer ID and not notarized, so Gatekeeper warns on first launch (right-click the app and choose **Open**). That is fine while the app only runs on our own Macs.

To ship it to other people later:

1. Join the Apple Developer Program and create a **Developer ID Application** certificate.
2. Store the certificate (`.p12`, base64) and its password as repository secrets, and import them into a temporary keychain in the workflow.
3. Set `PARALLAX_SIGN_IDENTITY` to the Developer ID identity so `build-app.sh` signs both `parallax-server` and the app, and enable the hardened runtime (`codesign --options runtime`).
4. Notarize with `xcrun notarytool submit --wait` using an App Store Connect API key stored as secrets, then `xcrun stapler staple Parallax.app` before zipping.
5. Note that ad hoc builds use the requirement `identifier "com.bddicken.parallax"`; switch the bundle ID to a Cloud Shuttle identifier before the first signed release.
