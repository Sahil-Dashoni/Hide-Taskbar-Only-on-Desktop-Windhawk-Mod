# Hide Taskbar Only on Desktop

A lightweight [Windhawk](https://windhawk.net/) mod that hides selected Windows taskbars when their corresponding display is showing only the desktop, and shows them again when an application or relevant Windows shell interaction requires the taskbar.

Each display is evaluated independently. An application on one display does not prevent a selected taskbar on another display from hiding when that display is showing only the desktop.

## Demo

### Multiple Displays

![Multiple Display](Assets/Multiple%20Display.gif)

Each selected display is evaluated independently. An application can keep one display's taskbar visible while another display remains in the desktop-only state.

### Single Display

![Single Display](Assets/Single%20Display.gif)

The taskbar hides when the display returns to the desktop and can be revealed by moving the cursor into the configured bottom-edge area.

## Features

- **Desktop-only taskbar hiding**
  - Hides a selected taskbar when its display has no visible, non-minimized application window.
  - Keeps the taskbar visible while an application is present.

- **Independent multi-monitor behavior**
  - Each selected display is evaluated separately.
  - Applications spanning multiple displays count on every display they intersect.
  - Maximized windows use the monitor assignment provided by Windows.

- **Bottom-edge hover reveal**
  - Reveals a taskbar hidden by the mod when the cursor enters the configured bottom area.
  - The reveal area automatically accounts for the taskbar's current height and display DPI.
  - An additional configurable margin can be added above the taskbar.
  - Hover reveal applies to bottom-docked taskbars.

- **Configurable hover-dismiss delay**
  - After a hover reveal, moving the cursor away starts a configurable countdown before the taskbar hides again.
  - Returning to the desktop by minimizing or closing the last application does not wait for this hover-dismiss delay.

- **Shell interaction handling**
  - Relevant Windows shell surfaces are treated as active UI instead of an empty desktop.
  - This includes supported Start menu, taskbar popup, tray/overflow, notification/Quick Settings, and Alt+Tab/task-switching situations.

- **Direct taskbar control**
  - The mod hides and shows the taskbar window directly.
  - It does not intentionally enable or change Windows' native taskbar auto-hide setting.

## How It Works

The main state-management logic runs in the mod's dedicated tool process. A small Explorer-side hook protects one specific secondary-taskbar visibility transition that can otherwise cause a brief flash.

For each selected display, the mod:

1. Discovers the taskbar associated with the display.
2. Enumerates relevant top-level windows and associates them with displays.
3. Determines whether a visible, non-minimized application window is present.
4. Accounts for supported Windows shell surfaces.
5. Checks the configured hover-reveal state.
6. Applies the resulting visibility state to that display's taskbar.

Taskbar ownership is tracked so the mod only restores taskbars that it actually hid.

## Difference from `taskbar-auto-hide-when-maximized`

Although both mods change when a taskbar is visible, they are designed around **different visibility rules**.

`taskbar-auto-hide-when-maximized` is primarily driven by application-window state. Its existing modes include conditions such as `intersected` and `maximized`.

This mod asks a different question:

> **Is this display showing only the desktop?**

If the answer is yes, the selected taskbar can hide. If any relevant visible, non-minimized application window is present on that display, the taskbar remains visible.

| Situation | This mod | `taskbar-auto-hide-when-maximized` |
| --- | --- | --- |
| Display is showing only the desktop | **Taskbar hides** | Not the purpose of its maximized/intersection modes |
| Normal non-maximized application is visible | **Taskbar remains visible** | Behavior depends on the selected window-state mode |
| Maximized application is visible | **Taskbar remains visible** | Designed to react to maximized/intersection state |
| One display has an app, another is idle | **Each selected display is evaluated independently** | Uses its own auto-hide/window-state policy |
| Main goal | **Taskbar hidden only on an idle desktop** | **Native auto-hide driven by window state** |

This distinction matters in everyday use. The goal here is not "hide the taskbar when a window is maximized." The goal is:

> **Keep the taskbar available while working, but remove it from a display when that display is completely on the desktop.**

The implementation is also intentionally different. This mod directly controls the taskbar window and keeps Windows' native taskbar auto-hide setting separate. It additionally provides its own per-display selection, desktop/application-state detection, shell handling, and configurable hover-reveal area.

If you specifically want Windows' native auto-hide behavior tied to maximized or taskbar-intersecting windows, `taskbar-auto-hide-when-maximized` is the more appropriate mod.

## Difference from Other Taskbar Auto-Hide Mods

### `taskbar-auto-hide-per-monitor`

This mod provides independent display selection too, but the focus here is different: this mod determines whether each selected display is currently desktop-only and directly controls taskbar visibility instead of changing Windows' native auto-hide state.

### `taskbar-auto-hide-custom-activation-area`

That mod changes the activation area used by Windows' native taskbar auto-hide. This mod instead determines when a display is desktop-only and provides its own bottom-edge hover-reveal behavior for taskbars hidden by the mod.

## Settings

### Extra hover margin

Adds extra space above the automatically detected taskbar-height reveal zone.

**Default:** `5 mm`

### Auto-hide delay after hover

Controls how long the taskbar remains visible after the cursor leaves the bottom reveal area.

**Default:** `700 ms`

This delay applies only to hover-based hiding. It does not delay hiding after the last application is minimized or closed.

### Taskbars to hide on desktop

Controls which displays use desktop-based taskbar hiding.

Displays can be selected individually or configured to apply to all displays.

Display selections use Windows display device identifiers such as `\\.\DISPLAY1`. These identifiers can differ from the display numbers shown in Windows Display Settings.

### Taskbars to reveal on hover

Controls which selected displays allow bottom-edge hover reveal.

Hover reveal is available only for bottom-docked taskbars.

## Multi-Monitor Example

Consider two displays:

1. An application is open on display 1.
2. Display 2 is showing only the desktop.
3. The selected taskbar on display 1 remains visible.
4. The selected taskbar on display 2 hides.
5. Moving the cursor into display 2's configured bottom-edge area reveals its taskbar.
6. Moving the cursor away starts the configured hover-dismiss delay.
7. Opening an application on display 2 keeps its taskbar visible.

The same logic is applied independently to every selected display.

## Windows Shell Interactions

The mod favors keeping the taskbar visible when supported Windows shell UI is active.

Supported shell handling includes relevant:

- Start menu surfaces
- Taskbar menus and popups
- Tray and notification overflow
- Notification and Quick Settings surfaces
- Alt+Tab and related task-switching UI

Shell surfaces are identified using relevant window classes together with the owning Windows shell process where required. Windows shell implementation details can change between Windows releases, so additional classes or processes may need to be added for future Windows versions.

## Explorer Integration

The Explorer-side component provides a narrow protection against a specific secondary-taskbar transition.

When a secondary taskbar has been hidden by this mod, Explorer may independently attempt to show it with `ShowWindow(..., SW_SHOWNA)`. The mod suppresses that specific transition only while the taskbar is still owned by a running instance of the dedicated tool process.

The protection:

- Applies only to the secondary taskbar.
- Applies only to a taskbar marked as hidden by this mod.
- Applies only to the `SW_SHOWNA` transition.
- Allows unrelated Explorer `ShowWindow` calls to proceed normally.

This prevents a brief secondary-taskbar flash without broadly intercepting Explorer window visibility operations.

### Failure Recovery

Taskbar ownership is associated with the dedicated tool process.

If that process terminates unexpectedly, stale ownership can be detected and the affected taskbar can be recovered instead of remaining permanently blocked by an old mod instance.

## Display and Explorer Changes

The mod refreshes taskbar and display state when relevant changes occur, including:

- Taskbar recreation
- Explorer-related state changes
- Display topology changes
- Monitor addition or removal
- Display configuration changes
- Windhawk setting changes

Taskbars are rediscovered rather than assuming that their window handles remain unchanged.

## Performance

The main application/display scan runs in the dedicated tool process rather than performing the full scan inside Explorer.

The mod uses:

- A dedicated worker thread for state management
- Event-driven refreshes for relevant changes
- A periodic safety poll for missed or unusual transitions
- A lightweight cursor-sampling thread for hover detection
- Adaptive cursor sampling when hover tracking is not required
- A one-shot timer for hover dismissal
- A narrow Explorer-side visibility hook for secondary-taskbar flash prevention

## Limitations

- Hover reveal is supported only for bottom-docked taskbars.
- Display identifiers such as `\\.\DISPLAY1` may differ from the numbering shown in Windows Display Settings.
- Display identifiers can change after display configuration changes.
- The current display-selection settings provide up to 16 `DISPLAYn` entries.
- The mod intentionally keeps Windows' native taskbar auto-hide setting separate from its own hiding behavior.
- Windows shell window classes and processes can change between Windows releases, so shell-interaction detection may require updates for future Windows versions.
- The mod is designed specifically around Windows Explorer/taskbar behavior and is not intended to be a general-purpose taskbar customization framework.

## Requirements

- Windows 10 or Windows 11
- [Windhawk](https://windhawk.net/)
- Windows Explorer shell (`explorer.exe`)

## Installation

### Install through Windhawk

1. Install [Windhawk](https://windhawk.net/).
2. Open Windhawk.
3. Go to **Home → Create a new mod**.
4. Paste the contents of `hide-taskbar-only-on-desktop.wh.cpp`.
5. Compile the mod.
6. Enable it.
7. Configure the displays and hover settings to your preference.

### Manual source installation

Clone or download this repository and use the `.wh.cpp` source file with Windhawk.

## Recommended Settings

The defaults are intended to provide a practical desktop-only taskbar experience:

```text
Extra hover margin:       5 mm
Auto-hide delay:          700 ms
Hide secondary taskbars:  Enabled
```

You can adjust these values from the Windhawk mod settings.

## Goal

The goal is a specific behavior:

> **Hide the taskbar when its display is showing only the desktop.**

The mod is not intended to replace Windows' native taskbar auto-hide or to be a general-purpose taskbar customization framework.

It is intended for users who want:

- A normal taskbar while working in applications
- A clean desktop when a display is idle
- Independent behavior across multiple displays
- Optional bottom-edge hover reveal
- Shell-aware taskbar visibility
- Windows' native auto-hide setting left untouched

## Author

**Sahil Dashoni**

GitHub: [@Sahil-Dashoni](https://github.com/Sahil-Dashoni)

## License

MIT License

Copyright (c) 2026 Sahil Dashoni

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
