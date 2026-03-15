This is a Claude Code plugin for running and testing Android apps. It defines a structured testing loop in `skills/running-android/SKILL.md` — that file owns the workflow. The three reference files (`GRADLE.md`, `BAZEL.md`, `ADB.md`) are linked from it and consulted for specific commands; they do not define workflow logic.

When modifying the testing loop, edit `SKILL.md`. When modifying command syntax or flags, edit the relevant reference file.
