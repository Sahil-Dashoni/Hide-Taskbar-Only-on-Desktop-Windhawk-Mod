# Hide Taskbar Only on Desktop

A lightweight [Windhawk](https://windhawk.net/) mod that automatically hides the Windows taskbar while you're on the desktop and brings it back when you need it.

The mod is designed to behave like a "desktop-only" taskbar: applications keep the taskbar available, while the desktop gets a clean, taskbar-free look.

## Features

- **Hide taskbar on desktop**
  - The taskbar automatically hides when no normal application window is active.
- **Show taskbar for applications**
  - The taskbar remains visible while using normal applications.
- **Bottom-edge hover reveal**
  - Move the mouse into the bottom area of the screen to reveal the taskbar.
  - The reveal zone automatically matches the current taskbar height.
  - An additional configurable margin can be added above the taskbar area.
- **Configurable hover hide delay**
  - After revealing the taskbar with the mouse, moving the pointer away starts a configurable countdown before the taskbar hides again.
- **Instant hide after minimizing/closing the last window**
  - The hover delay is not used when returning to the desktop by minimizing or closing the last active application.
- **Taskbar interaction**
  - Clicking taskbar buttons works normally.
  - The mod does not treat the taskbar itself as the desktop, so applications can still be focused normally.
- **Start menu and shell UI support**
  - Start menu and supported Explorer shell UI are treated as active UI instead of the empty desktop.
- **Multiple monitors**
  - Secondary taskbars can also be hidden and shown when Windows is configured to show taskbars on all displays.

## How It Works

The mod runs inside `explorer.exe` and periodically checks the current Windows state.

It determines whether:

1. A normal application window is active.
2. The desktop is the active state.
3. The mouse is inside the bottom reveal area.
4. The taskbar was revealed specifically by mouse hover.

The taskbar is then shown or hidden according to those states.

### Behavior

| Situation | Taskbar |
|---|---|
| Desktop, mouse outside reveal area | Hidden |
| Desktop, mouse inside reveal area | Visible |
| Mouse leaves reveal area after hover | Hides after configured delay |
| Application active | Visible |
| Last application minimized/closed | Hides immediately |
| Taskbar button clicked | Works normally |
| Start menu / supported shell UI open | Visible |

## Settings

The mod currently provides three settings.

### Extra hover margin

Adds extra space above the automatically detected taskbar-height reveal zone.

**Default:** `5 mm`

This means the mouse does not have to be positioned precisely at the taskbar boundary.

### Auto-hide delay after hover

Controls how long the taskbar remains visible after the mouse leaves the bottom reveal area.

**Default:** `700 ms`

This delay applies **only** when the taskbar was revealed by hovering near the bottom.

It does **not** delay hiding after minimizing or closing the last application.

### Hide secondary taskbars

Controls whether taskbars on additional monitors are also hidden.

**Default:** `true`

## Installation

### Install through Windhawk

1. Install [Windhawk](https://windhawk.net/).
2. Open Windhawk.
3. Go to **Home** → **Create a new mod**.
4. Paste the contents of:
   `hide-taskbar-only-on-desktop.wh.cpp`
5. Compile the mod.
6. Enable it.

### Manual source installation

Clone or download this repository and use the `.wh.cpp` source file with Windhawk.

## Recommended Settings

The defaults are intended to work well for a normal Windows 10/11 setup:

```text
Extra hover margin:       5 mm
Auto-hide delay:          700 ms
Hide secondary taskbars:  Enabled
```

You can adjust these values from the Windhawk mod settings.

## Requirements

- Windows 10 or Windows 11
- [Windhawk](https://windhawk.net/)
- Explorer shell (`explorer.exe`)

## Known Behavior

The mod intentionally favors keeping the taskbar visible whenever the Windows shell state is ambiguous. This helps avoid unexpectedly hiding the taskbar during Explorer transitions or while interacting with shell UI.

The hover area is based on the taskbar's current window height, so it adapts automatically to the taskbar size and Windows display scaling.

## File

```text
hide-taskbar-only-on-desktop.wh.cpp
```

## Author

**Sahil Dashoni**

GitHub: [@Sahil-Dashoni](https://github.com/Sahil-Dashoni)

## License

MIT License

Copyright (c) 2026 Sahil Dashoni

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files, to deal in the Software
without restriction, including without limitation the rights to use, copy,
modify, merge, publish, distribute, sublicense, and/or sell copies of the
Software, and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
