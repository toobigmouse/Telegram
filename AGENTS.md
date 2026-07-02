# Telegram for Android – Agent Guide

Fork of [DrKLO/Telegram](https://github.com/DrKLO/Telegram). Official Telegram source; the `APP_ID` and `APP_HASH` in BuildVars.java are placeholders — replace before running.

## Build commands

```bash
# Full debug build (all flavors)
./gradlew :TMessagesProj_App:assembleBundleAfatDebug

# Release APK
./gradlew :TMessagesProj_App:assembleAfatRelease

# Release AAB
./gradlew :TMessagesProj_App:bundleBundleAfatRelease

# Standalone APK (non-Play-Store variant)
./gradlew :TMessagesProj_AppStandalone:assembleAfatStandalone

# Huawei variant
./gradlew :TMessagesProj_AppHuawei:assembleAfatRelease

# Instrumented tests only (no unit tests exist — only androidTest/)
./gradlew :TMessagesProj_AppTests:connectedDebugAndroidTest
```

## Project structure

| Module | Type | Purpose |
|---|---|---|
| `:TMessagesProj` | Android library | All app code (Java), native JNI via CMake, resources |
| `:TMessagesProj_App` | App | Main Google-Play entry point |
| `:TMessagesProj_AppHuawei` | App | Huawei variant (HMS instead of GMS) |
| `:TMessagesProj_AppHockeyApp` | App | AppCenter beta distribution variant |
| `:TMessagesProj_AppStandalone` | App | Standalone (no Google Play) |
| `:TMessagesProj_AppTests` | Test APK | Instrumented tests only, written in Kotlin |

## Key toolchain versions

- **AGP**: 8.6.1
- **Gradle**: 8.7
- **Kotlin**: 1.9.20
- **NDK**: 27.2.12479018 (`local.properties` has 21.4.7075529 for local builds)
- **CMake**: 3.10.2
- **Java**: 1.8 (source & target)
- **SDK**: compile 35, min 21 (23 for `bundleAfat_SDK23` flavor), target 35

## Build configuration quirks

- **All release signing uses `TMessagesProj/config/release.keystore`** with dummy passwords (`android` for everything) — not for production.
- **Build types**: `debug`, `release`, `standalone`, `HA_private`, `HA_public`, `HA_hardcore` (the HA* types are for HockeyApp beta builds).
- **Flavors**: `afat` (APK, all ABIs), `bundleAfat` (AAB, minSdk 21), `bundleAfat_SDK23` (AAB, minSdk 23).
- Debug builds get `applicationIdSuffix ".beta"`.
- `variantFilter` ignores non-release non-afat combos.
- R8 full mode enabled (`android.enableR8.fullMode=true`).
- `coreLibraryDesugaringEnabled true` (desugar_jdk_libs 2.1.5).
- The `checkVisibility` Gradle task runs before every build and blocks building public variants from a private-config fork.
- Firebase google-services.json exists per module (dummy files — replace for your own Firebase project).

## Prerequisites before first build

1. Obtain your own `api_id` and `api_hash` from https://core.telegram.org/api/obtaining_api_id
2. Set them in `TMessagesProj/src/main/java/org/telegram/messenger/BuildVars.java:29-30`
3. (Optional) Replace `google-services.json` files with your Firebase project's config
4. (Optional) Replace `TMessagesProj/config/release.keystore` with your own keystore

## Testing

- Only **instrumented tests** (`androidTest/`) — no JVM unit tests.
- Written in Kotlin with JUnit 4 runner (`androidx.test.runner.AndroidJUnitRunner`).
- Tests live in `TMessagesProj_AppTests` module under `org.telegram.tgnet.test`.
- Run on emulator/device: `./gradlew :TMessagesProj_AppTests:connectedDebugAndroidTest`

## Reproducible builds

- `apkdiff.py <apk1> <apk2>` — binary-identical APK comparison (ignoring signing certs).
- `apkfrombundle.py <aab> <apk>` — verify APK extracted from AAB matches standalone build.

## Native code

JNI code in `TMessagesProj/jni/` with `CMakeLists.txt`. Builds are triggered automatically by the Gradle NDK integration.
