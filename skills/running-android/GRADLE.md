# Gradle Reference

Use `./gradlew` (or `gradlew.bat` on Windows) from the project root.

---

## Common Tasks

| Task | Command |
|---|---|
| List all tasks | `./gradlew tasks` |
| Assemble debug APK | `./gradlew assembleDebug` |
| Assemble release APK | `./gradlew assembleRelease` |
| Install debug to device | `./gradlew installDebug` |
| Uninstall | `./gradlew uninstallDebug` |
| Run unit tests | `./gradlew test` |
| Run instrumented tests | `./gradlew connectedAndroidTest` |
| Clean | `./gradlew clean` |

---

## Build Variants

Android projects support build types (`debug`, `release`) and product flavors. Variants combine them (e.g., `freeDebug`, `paidRelease`).

To list all variants for a module:
```
./gradlew :app:tasks --group="build"
```

To assemble a specific variant:
```
./gradlew :app:assembleFreeDebug
```

To install a specific variant:
```
./gradlew :app:installFreeDebug
```

---

## Multi-Module Projects

Prefix tasks with the module name using `:moduleName:`:

```
./gradlew :feature:login:assembleDebug
./gradlew :app:installDebug
```

To see all subprojects:
```
./gradlew projects
```

---

## APK Output Paths

After assembling, APKs are written to:

```
<module>/build/outputs/apk/<variant>/<module>-<variant>.apk
```

Examples:
```
app/build/outputs/apk/debug/app-debug.apk
app/build/outputs/apk/release/app-release-unsigned.apk
app/build/outputs/apk/freeDebug/app-free-debug.apk
```

The exact filename depends on the `archivesBaseName` setting in `build.gradle`. When in doubt, list the directory:
```
ls app/build/outputs/apk/
```

---

## Unit Tests

Run JVM unit tests (no device required):
```
./gradlew test
./gradlew :app:testDebugUnitTest
```

Results: `app/build/reports/tests/testDebugUnitTest/index.html`

---

## Instrumented Tests (Connected Tests)

Requires a connected device or emulator:
```
./gradlew connectedAndroidTest
./gradlew :app:connectedDebugAndroidTest
```

Results: `app/build/reports/androidTests/connected/index.html`

---

## Gradle Wrapper Version

Check the wrapper version:
```
./gradlew --version
```

Update the wrapper:
```
./gradlew wrapper --gradle-version=<version>
```
