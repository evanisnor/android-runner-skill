This is a Claude Code plugin that guides Claude through a human-in-the-loop Android testing workflow. When installed in an Android project, it teaches Claude how to build an app, get it onto a device, and iterate with a human tester — rather than assuming the testing is automated.

The plugin contains one skill (`running-android`) backed by three reference files covering Gradle, Bazel, and ADB.

## The testing loop

The core of this plugin is the 10-step loop in `skills/running-android/SKILL.md`:

1. Detect build system (Gradle vs Bazel)
2. Identify the target/module to build
3. Build the APK
4. Select a connected device via `adb devices`
5. Install the APK
6. Launch the app
7. **Ask the human what to test** — this is where Claude hands off to the human
8. Wait for feedback
9. Optionally collect logcat or take a screenshot
10. Iterate: fix → rebuild → reinstall → relaunch, or declare done

Step 7 is intentional. Claude should not proceed past launch without asking the human what to verify.

## Skills and reference files

`SKILL.md` is the entry point and owns the workflow. The three reference files are linked from it and consulted for specific commands:

- **`GRADLE.md`** — `./gradlew` tasks, build variants, APK output paths (`app/build/outputs/apk/<variant>/`), unit vs instrumented tests
- **`BAZEL.md`** — target discovery via `bazel query`, `bazel build`, `bazel mobile-install --start_app`, APK location under `bazel-bin/`
- **`ADB.md`** — device selection with `-s <serial>`, `adb install -r`, `am start`, logcat filters, `pm clear`, screencap

## Trigger

The skill auto-triggers when the user says things like "run the app", "install the APK", "test on device", or "build and deploy". It can also be invoked manually with `/android-runner:running-android`.
