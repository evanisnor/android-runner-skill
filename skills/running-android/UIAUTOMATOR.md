# UIAutomator Reference

Commands for driving and inspecting the Android UI autonomously via ADB.

See also: [SKILL.md](SKILL.md)

---

## UI Hierarchy Dump

Dump the current UI hierarchy to a file on the device, then pull it locally:

```sh
adb shell uiautomator dump /sdcard/ui.xml
adb pull /sdcard/ui.xml
```

The dump implicitly waits for the UI thread to be idle before capturing. Add a short `sleep 1` after performing an input action before dumping, to let animations settle:

```sh
adb shell input tap 540 960 && sleep 1 && adb shell uiautomator dump /sdcard/ui.xml && adb pull /sdcard/ui.xml
```

### Reading the hierarchy

Each element in `ui.xml` carries attributes relevant to validation:

| Attribute | What it tells you |
|-----------|-------------------|
| `resource-id` | Stable identifier (e.g. `com.example:id/login_button`) |
| `text` | Visible label or field content |
| `content-desc` | Accessibility description |
| `bounds` | Position as `[left,top][right,bottom]` |
| `enabled` | Whether the element can be interacted with |
| `focused` | Whether the element currently has focus |
| `selected` | Whether the element is in a selected/checked state |
| `clickable` | Whether the element responds to taps |
| `scrollable` | Whether the element is a scrollable container |

Use `resource-id` and `text` as the primary signals for pass/fail criteria.

---

## Computing Tap Coordinates from Bounds

Given `bounds="[left,top][right,bottom]"`, the center tap point is:

```
x = (left + right) / 2
y = (top + bottom) / 2
```

Example: `bounds="[100,400][980,520]"` → tap at `(540, 460)`.

---

## Input: Tap

```sh
adb shell input tap <x> <y>
```

Example:
```sh
adb shell input tap 540 960
```

---

## Input: Swipe / Scroll

```sh
adb shell input swipe <x1> <y1> <x2> <y2> [duration_ms]
```

Scroll down (swipe upward):
```sh
adb shell input swipe 540 1200 540 400 300
```

Scroll up (swipe downward):
```sh
adb shell input swipe 540 400 540 1200 300
```

---

## Input: Text Entry

```sh
adb shell input text "string"
```

Spaces must be URL-encoded as `%s`:
```sh
adb shell input text "hello%sworld"
```

Tap the target field first to focus it before entering text.

---

## Input: Key Events

```sh
adb shell input keyevent <code>
```

Common key codes:

| Key | Code |
|-----|------|
| BACK | 4 |
| HOME | 3 |
| ENTER | 66 |
| TAB | 61 |
| DEL (backspace) | 67 |
| DPAD_UP | 19 |
| DPAD_DOWN | 20 |
| DPAD_LEFT | 21 |
| DPAD_RIGHT | 22 |

---

## Current Activity

Identify which activity (and fragment, if logged) is currently in the foreground:

```sh
adb shell dumpsys activity activities | grep mResumedActivity
```

Use this to confirm navigation reached the expected screen before dumping the hierarchy.

---

## Screenshot

Capture the screen and pull it locally for visual inspection:

```sh
adb exec-out screencap -p > screen.png
```

Take a screenshot after each significant action to track state progression.

---

## With Multiple Devices

When multiple devices are connected, pass `-s <serial>` to all commands:

```sh
adb -s <serial> shell uiautomator dump /sdcard/ui.xml
adb -s <serial> pull /sdcard/ui.xml
adb -s <serial> shell input tap 540 960
```
