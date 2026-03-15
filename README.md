# android-runner-skill

A Claude Code plugin for building, installing, and testing Android apps. Supports Gradle and Bazel build systems with ADB device management. Includes a structured testing loop where Claude attempts autonomous UI validation first and hands off to the human only when stuck.

## Installation

Install the plugin by pointing Claude Code at this directory:

```
claude --plugin-dir /path/to/android-runner-skill
```

Or add it to your Claude Code settings to load it automatically for Android projects.

## Usage

### Auto-trigger

The skill triggers automatically when you ask Claude to do things like:

- "Run the app on my device"
- "Install the APK"
- "Test this on an emulator"
- "Build and deploy to device"
- "Debug on Android"

### Manual trigger

```
/android-runner:running-android
```

## What it does

The `running-android` skill walks through a structured testing loop:

1. Detects your build system (Gradle or Bazel)
2. Identifies the build target or module
3. Builds the APK
4. Selects a connected device or emulator
5. Installs and launches the app
6. Extracts test criteria from your task description
7. Attempts autonomous validation — dumps the UI hierarchy, reads it, taps/swipes/types, checks logcat, and reports pass or fail
8. Falls back to you only if it gets stuck or the criteria are ambiguous
9. Iterates — rebuilds, reinstalls, and reruns as needed

## Reference

| File | Contents |
|---|---|
| `skills/running-android/GRADLE.md` | `./gradlew` commands, build variants, APK paths, test tasks |
| `skills/running-android/BAZEL.md` | `bazel build`, `mobile-install`, target discovery |
| `skills/running-android/ADB.md` | Device selection, install, launch, logcat, screenshots |
| `skills/running-android/UIAUTOMATOR.md` | UI hierarchy dump, tap/swipe/text/keyevent input, bounds parsing |

## Requirements

- `adb` must be on your PATH (from Android SDK platform-tools)
- For Gradle projects: a valid `gradlew` wrapper in the project root
- For Bazel projects: `bazel` or `bazelisk` on your PATH
- For APK inspection: `aapt` or `apkanalyzer` (from Android SDK build-tools / cmdline-tools)
