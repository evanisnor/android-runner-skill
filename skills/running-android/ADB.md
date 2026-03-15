# ADB Reference

`adb` (Android Debug Bridge) communicates with connected devices and emulators.

---

## Device Management

List connected devices and emulators:
```
adb devices
adb devices -l   # includes device details
```

Target a specific device when multiple are connected (use `-s` before the subcommand):
```
adb -s <serial> install app.apk
adb -s emulator-5554 shell am start ...
```

Get the serial of the only connected device:
```
adb get-serialno
```

Wait for a device to come online:
```
adb wait-for-device
```

---

## Installing APKs

Standard install:
```
adb install app-debug.apk
```

Replace an existing installation (`-r`):
```
adb install -r app-debug.apk
```

Allow test APKs (`-t`):
```
adb install -t -r app-debug.apk
```

Install to a specific device:
```
adb -s <serial> install -r app-debug.apk
```

Uninstall by package name:
```
adb uninstall com.example.myapp
```

---

## Launching the App

Start the main activity:
```
adb shell am start -n <package>/<activity>
```

Example:
```
adb shell am start -n com.example.myapp/.MainActivity
```

Start with a specific intent action:
```
adb shell am start -a android.intent.action.VIEW -d "https://example.com" com.example.myapp
```

Stop the app:
```
adb shell am force-stop com.example.myapp
```

---

## Clearing App State

Clear all app data (equivalent to uninstall + reinstall):
```
adb shell pm clear com.example.myapp
```

Clear only shared preferences or databases: connect via shell and delete files manually:
```
adb shell
run-as com.example.myapp
rm -rf /data/data/com.example.myapp/shared_prefs/
```

---

## Logcat

Stream all logs:
```
adb logcat
```

Filter by tag:
```
adb logcat -s MyTag:D
```

Filter by package (Android 7+):
```
adb logcat --pid=$(adb shell pidof -s com.example.myapp)
```

Filter by log level (`V`, `D`, `I`, `W`, `E`):
```
adb logcat *:E          # errors only
adb logcat *:W          # warnings and above
```

Useful combined filter (suppress noise, show app logs):
```
adb logcat -s MyAppTag:V AndroidRuntime:E
```

Clear the log buffer before a test run:
```
adb logcat -c
```

Save logcat to a file:
```
adb logcat -d > logcat.txt
```

---

## Screenshots

Capture a screenshot on device and pull it:
```
adb shell screencap -p /sdcard/screen.png
adb pull /sdcard/screen.png
adb shell rm /sdcard/screen.png
```

One-liner:
```
adb exec-out screencap -p > screen.png
```

---

## Screen Recording

Record up to 3 minutes of video:
```
adb shell screenrecord /sdcard/demo.mp4
adb pull /sdcard/demo.mp4
adb shell rm /sdcard/demo.mp4
```

---

## Getting Package and Activity Names

From an installed APK using `aapt` (Android SDK build tools):
```
aapt dump badging app-debug.apk | grep -E "package:|launchable-activity"
```

From an installed APK using `apkanalyzer` (Android SDK cmdline-tools):
```
apkanalyzer manifest application-id app-debug.apk
apkanalyzer manifest print app-debug.apk | grep -E "package|activity"
```

From a device (list installed packages):
```
adb shell pm list packages | grep myapp
```

Get the current foreground activity (useful to confirm launch):
```
adb shell dumpsys activity activities | grep mResumedActivity
```

---

## File Transfer

Push a file to the device:
```
adb push local-file.txt /sdcard/local-file.txt
```

Pull a file from the device:
```
adb pull /sdcard/some-file.txt
```

---

## Shell Access

Open an interactive shell:
```
adb shell
```

Run a single command:
```
adb shell <command>
```
