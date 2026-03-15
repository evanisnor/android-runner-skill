# Bazel Reference

Bazel is identified by the presence of `WORKSPACE`, `WORKSPACE.bazel`, or `MODULE.bazel` in the project root.

---

## Target Discovery

Find all Android binary targets in the project:
```
bazel query 'kind(android_binary, //...)'
```

Find all Android test targets:
```
bazel query 'kind(android_instrumentation_test, //...)'
```

List all targets in a specific package:
```
bazel query //app:all
```

Inspect a target's attributes:
```
bazel query --output=build //app:app
```

---

## Building

Build an Android binary:
```
bazel build //app:app
```

Build with a specific configuration:
```
bazel build --config=release //app:app
```

Build all targets:
```
bazel build //...
```

---

## APK Location

After a successful build, APKs are written under `bazel-bin/`:
```
bazel-bin/app/app.apk
```

The path mirrors the target label: `//app:app` → `bazel-bin/app/app.apk`.

To find the APK output for a target:
```
bazel cquery --output=files //app:app
```

---

## Install and Launch with mobile-install

`bazel mobile-install` builds, installs, and optionally starts the app in a single step. It uses incremental deployment to speed up iteration.

Install only:
```
bazel mobile-install //app:app
```

Install and start the app:
```
bazel mobile-install //app:app --start_app
```

Target a specific device (when multiple are connected):
```
bazel mobile-install //app:app --start_app --adb_arg=-s --adb_arg=<serial>
```

---

## Running Tests

Run all tests:
```
bazel test //...
```

Run a specific instrumented test target:
```
bazel test //app:app_tests
```

Run with verbose output:
```
bazel test //app:app_tests --test_output=all
```

---

## Useful Flags

| Flag | Purpose |
|---|---|
| `--config=<name>` | Use a named config from `.bazelrc` |
| `--define=<key>=<value>` | Pass build-time variables |
| `--compilation_mode=opt` | Optimized (release-like) build |
| `--compilation_mode=dbg` | Debug build |
| `--verbose_failures` | Show full command on failure |
| `--sandbox_debug` | Debug sandbox issues |

---

## Inspecting the Build Graph

Show what a target depends on:
```
bazel query 'deps(//app:app)'
```

Show what depends on a target:
```
bazel query 'rdeps(//..., //lib:mylib)'
```

---

## Bazel Version

```
bazel version
```

Check `.bazelversion` in the project root for the pinned version (used by Bazelisk).
