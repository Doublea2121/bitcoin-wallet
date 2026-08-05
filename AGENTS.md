# Agent Guidance for bitcoin-wallet

This repository contains an Android wallet app under `wallet/` and app store metadata under `metadata/`.

## Recommended callouts for AI agents

- `wallet/` is the main codebase. Most source and resource changes happen under `wallet/src/`, `wallet/res/`, `wallet/res-prod/`, `wallet/assets/`, and `wallet/assets-prod/`.
- `wallet/build.gradle` defines Android SDK settings, flavors, packaging options, and key dependencies.
- `settings.gradle` enforces Gradle version compatibility: Gradle 4.4 <= version < 7.0.
- `build.gradle` (root) pins the Android Gradle plugin and top-level project settings.
- `wallet/README.md` is the best source for Android-specific build, install, and translation workflow details.

## Build and test commands

- `gradle clean build`
- `gradle clean test :wallet:assembleDevDebug`
- `gradle :wallet:installDevDebug`
- `gradle clean test :wallet:assembleProdRelease`
- `buildah build --cap-add sys_admin --device /dev/fuse --file build.Containerfile --output build/ .`

## Important conventions

- This repo does not include a Gradle wrapper. Use a system-installed Gradle matching `settings.gradle`.
- The Android app uses two product flavors: `dev` (Testnet, `.wallet_test`) and `prod` (Mainnet, `.wallet`).
- Resources for `prod` are overridden in `res-prod/` and `assets-prod/`.
- `wallet/build.gradle` contains an extensive `packagingOptions` block; check it before adding libraries that may add unmanaged `META-INF` or protobuf assets.
- Java compatibility is set to Java 8, but the build requires Java 11 as the host SDK.

## Checks after changes

- If you modify production-only resources or packaging, verify `:wallet:assembleProdRelease`.
- For most code changes, `gradle clean test :wallet:assembleDevDebug` is the practical verification command.
- Use `adb logcat` and device file paths from `wallet/README.md` for runtime debugging.

## Useful references

- `README.md` — repo-level prerequisites and reproducible build information.
- `wallet/README.md` — Android developer workflow, install, and translation notes.
- `.github/copilot-instructions.md` — additional agent-specific behavior guidance.
