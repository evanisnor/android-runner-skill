---
description: Guides Claude through running, installing, and testing an Android app on a physical device or emulator. Triggers when the user wants to run the app, test on device, install an APK, build and deploy, debug on Android, or iterate through a manual testing loop with human feedback. Covers Gradle and Bazel build systems with ADB device management.
---

# Running Android

Use this skill when the user wants to build, install, or test an Android app on a device or emulator. This skill defines a structured testing loop that includes the human in the feedback cycle.

For build system commands, see:
- [GRADLE.md](GRADLE.md) — `./gradlew` commands, build variants, APK paths
- [BAZEL.md](BAZEL.md) — `bazel build`, `bazel mobile-install`, target discovery
- [ADB.md](ADB.md) — device selection, install, launch, logcat, screenshots
- [UIAUTOMATOR.md](UIAUTOMATOR.md) — UI hierarchy dump, input tap/swipe/text, key events

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

### 7. Extract Test Criteria

Before interacting with the app, derive criteria from the user's task description:
- What feature or behavior is under test?
- What UI states, elements, or text are expected to appear?
- What actions should be taken to reach the testable state?
- What constitutes a clear pass? A clear fail?

If the task description is clear enough to answer these, proceed to step 8 autonomously. If criteria are ambiguous or the task description doesn't describe observable UI outcomes, ask the user to clarify before proceeding.

### 8. Autonomous Validation Loop

Use the criteria from step 7 to drive the app. See UIAUTOMATOR.md for all commands.

1. Take a screenshot (`adb exec-out screencap -p > screen.png`) and read it visually to orient.
2. Dump the UI hierarchy (`adb shell uiautomator dump /sdcard/ui.xml` + `adb pull /sdcard/ui.xml`) and inspect element attributes for expected content.
3. Perform input actions (tap, swipe, text entry) to navigate toward the test scenario.
4. After each action, take a screenshot and/or re-dump the UI hierarchy to verify state progressed as expected.
5. Check logcat for relevant output if the criteria involve behavior not visible in the UI.
6. Evaluate against criteria:
   - **Pass**: expected elements or text are present and no errors found — report results and go to step 10.
   - **Fail**: unexpected state or errors found — collect screenshot and logcat, report to user, go to step 10 to iterate.
   - **Stuck**: UI state is unclear, an action had no effect, the navigation path is unknown, or criteria can't be verified from available signals — fall through to step 9.

Keep the loop to ≤ 10 actions before declaring stuck.

### 9. Human Handoff (fallback)

Triggered only when step 8 gets stuck or criteria were ambiguous in step 7.

Report what was attempted and what was unclear, then ask:

> "I wasn't able to verify [criteria] autonomously because [reason]. The app is running on [device]. Can you take over and tell me what you observe?"

Wait for feedback. The human may:
- Report a bug or unexpected behavior
- Ask for logs or a screenshot
- Ask for a code change
- Confirm the behavior looks correct

### 10. Iterate

Based on results from step 8 or feedback from step 9:
- **Bug to fix**: make the code change, rebuild, reinstall, and relaunch. Return to step 3.
- **Test something else**: return to step 7 with new criteria.
- **Done**: summarize what was tested, what passed, what failed (if anything), and what was confirmed by autonomous vs. human verification.

Before rebuilding after a code change, confirm the change with the user.
