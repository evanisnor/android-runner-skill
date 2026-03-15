---
description: Guides Claude through running, installing, and testing an Android app on a physical device or emulator. Triggers when the user wants to run the app, test on device, install an APK, build and deploy, debug on Android, or iterate through a manual testing loop with human feedback. Covers Gradle and Bazel build systems with ADB device management.
---

# Running Android

Use this skill when the user wants to build, install, or test an Android app on a device or emulator. This skill defines a structured testing loop that includes the human in the feedback cycle.

For build system commands, see:
- [GRADLE.md](GRADLE.md) — `./gradlew` commands, build variants, APK paths
- [BAZEL.md](BAZEL.md) — `bazel build`, `bazel mobile-install`, target discovery
- [ADB.md](ADB.md) — device selection, install, launch, logcat, screenshots

---

## Testing Loop

Follow these steps in order. Do not skip steps. Ask the user if anything is ambiguous.

### 1. Detect Build System

Check the project root for:
- `gradlew` or `gradlew.bat` → Gradle project, see GRADLE.md
- `WORKSPACE`, `WORKSPACE.bazel`, or `MODULE.bazel` → Bazel project, see BAZEL.md

If both exist, ask the user which build system to use.

### 2. Identify Target or Module

For Gradle:
- If there is a single `app/` module, default to it.
- If multiple modules exist, ask the user which one to build.
- Ask for the build variant if it is not obvious (default to `debug`).

For Bazel:
- Run `bazel query 'kind(android_binary, //...)'` to discover targets.
- If there is only one result, use it. If multiple, ask the user.

### 3. Build

Run the appropriate build command for the detected build system and target. Show the user the command before running it.

If the build fails:
- Show the relevant error output.
- Attempt to diagnose and fix if the cause is clear.
- Ask the user before making changes to build files.

### 4. Select Device

Run `adb devices` and list connected devices and emulators.

- If one device is connected, use it automatically.
- If multiple devices are connected, ask the user to choose.
- If no devices are connected, tell the user and wait.

### 5. Install APK

Install the built APK to the selected device. Use the `-r` flag to replace an existing installation.

For Bazel with `mobile-install`, the install step may be combined with build — check BAZEL.md.

### 6. Launch App

Start the app using `adb shell am start`. You will need the package name and main activity.

- For Gradle: check `AndroidManifest.xml` or `build.gradle` for `applicationId`.
- For Bazel: check the `android_binary` target's `manifest` attribute.
- To extract from the APK directly: see ADB.md for `aapt` and `apkanalyzer` usage.

### 7. Tell the Human What to Test

Do not assume what the human should verify. Explicitly ask:

> "The app is running on [device]. What would you like to test or verify?"

Or, if the user already described a scenario, summarize the test steps and ask them to proceed.

### 8. Wait for Feedback

Wait for the human to test and report back. Do not proceed until they respond. They may:
- Report a bug or unexpected behavior
- Ask you to collect logs
- Ask you to make a code change
- Say the behavior looks correct

### 9. Optionally Collect Logcat

If the user reports an issue or asks for logs, collect logcat output. See ADB.md for filter patterns.

Offer to take a screenshot if a visual issue is reported. See ADB.md for screencap usage.

### 10. Iterate

Based on feedback:
- **Bug to fix**: make the code change, rebuild, reinstall, and relaunch. Return to step 3.
- **Want to test something else**: return to step 7.
- **Done**: summarize what was tested and confirmed.

Before rebuilding after a code change, confirm with the user that the change looks correct.
