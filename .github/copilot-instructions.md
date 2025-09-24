<!--
Guidance for AI coding agents working on the bitcoin-wallet repository.
Keep this file short, focused, and actionable. Update it only with discoverable
project facts (build commands, file locations, conventions) — not policies.
-->

# Copilot / AI Agent Instructions (bitcoin-wallet)

Purpose: quickly orient an AI agent to be productive editing, building and
testing the Android `wallet` app in this repository.

- **Big picture**: this repo holds two main parts:
  - `wallet/` — the Android app (Java, Gradle, flavors `dev` and `prod`).
  - `metadata/` — app store descriptions and translations.

- **Why structure is this way**: a single Gradle multi-project build keeps
  Android sources and store metadata together. Product flavors (`dev`/`prod`)
  control runtime differences (Testnet/dev vs Mainnet/prod, different
  `applicationIdSuffix`, assets/res overrides in `assets-prod`/`res-prod`).

- **Key files to open first**:
  - `wallet/build.gradle` — Android SDK, flavors, packaging options, key
    dependencies (bitcoinj, OkHttp, Moshi, Room).
  - `settings.gradle` — enforces Gradle version range (4.4 <= Gradle < 7.0).
  - `build.gradle` (root) — Gradle plugin versions (Android plugin 3.1.4).
  - `wallet/AndroidManifest.xml`, `wallet/src/` and `wallet/res/` — app code.

- **Build / test commands (copyable)**:
  - Full build: `gradle clean build`
  - Dev APK: `gradle clean test :wallet:assembleDevDebug`
  - Install dev APK to connected device: `gradle :wallet:installDevDebug`
  - Prod APK (unsigned): `gradle clean test :wallet:assembleProdRelease`
  - Containerized reproducible build: `buildah build --cap-add sys_admin --device /dev/fuse --file build.Containerfile --output build/ .`

  - Windows / PowerShell notes:
    - Prefer the wrapper for reproducible builds on Windows: `.\gradlew.bat clean test :wallet:assembleDevDebug`
    - Install + run on a connected device: `.\gradlew.bat :wallet:installDevDebug`
    - `gradle` may work if installed system-wide, but `.\gradlew.bat` is recommended.

- **Runtime / debug notes**:
  - Development flavor uses Testnet and produces wallet file at
    `/data/data/de.schildbach.wallet_test/files/wallet-protobuf-testnet`.
  - Mainnet wallet paths live under `/data/data/de.schildbach.wallet/files/...`.
  - Useful device commands: `adb logcat` (logs), `adb pull <path>` (pull wallet file).

- **Tests**: unit tests use JUnit (`testImplementation 'junit:junit:4.13.2'`).
  Run `gradle test` or `gradle clean test` to execute JVM/unit tests.

- **Project conventions & patterns** (do not assume defaults):
  - Gradle version is constrained in `settings.gradle`; ensure the runtime
    Gradle matches that range before running tasks.
  - The Android Gradle plugin is pinned (`com.android.tools.build:gradle:3.1.4`),
    so avoid making sweeping buildscript/plugin upgrades without manual review.
  - Flavor-specific resources: `res-prod` and `assets-prod` provide overrides
    for the `prod` flavor — keep both in sync when changing resources.
  - `packagingOptions` in `wallet/build.gradle` excludes many META-INF and
    third-party files; when adding libraries that add resource files, check
    this list if packaging failures appear.
  - Custom Gradle task `svgToPngMipmap` converts `graphics/mipmap` SVGs to PNG
    resources — run or inspect when graphics/mipmap changes.

- **Integration points / external services**:
  - Bitcoin logic uses `org.bitcoinj:bitcoinj-core:0.16.5` — inspect usages in
    `wallet/src` for network, wallet, and address handling patterns.
  - Exchange rates are fetched from CoinGecko (`https://api.coingecko.com/api/v3/exchange_rates`) and gated by compile-time flag `Constants.ENABLE_EXCHANGE_RATES`.
  - Translations are managed via Transifex (see `wallet/README.md`); `tx pull` and `tx push` commands are used by maintainers.

- **When you change code** (practical checklist for an AI agent):
  1. Run `gradle clean test :wallet:assembleDevDebug` locally (or equivalent CI task).
  2. If you touch resources used only in `prod`, verify `:wallet:assembleProdRelease`.
  3. Check `wallet/build/outputs/apk/` for generated APK paths.
  4. If adding native or third-party resources, re-check `packagingOptions`.

- **Search tips (where to look for examples)**:
  - Wallet bitcoin usage: search `wallet/src/**` for `bitcoinj` imports.
  - Flavor overrides: compare `res/` vs `res-prod/` and `assets/` vs `assets-prod/`.
  - Build logic: inspect `settings.gradle`, `build.gradle` (root) and `wallet/build.gradle`.

Examples (representative files using `bitcoinj`):
- `wallet/src/de/schildbach/wallet/service/BlockchainService.java` — core peer/network/blockchain handling.
- `wallet/src/de/schildbach/wallet/WalletApplication.java` — bitcoinj `Context` and wallet initialization.
- `wallet/src/de/schildbach/wallet/util/WalletUtils.java` — wallet (de)serialization and helpers.

If anything here is unclear or you need additional examples (CI commands,
emulator setup, or common test failures), tell me which area to expand.
