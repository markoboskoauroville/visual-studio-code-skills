---
name: terminal-control
description: Open, place, focus, and use a visible Terminal window through Macro Recorder, especially on the MacBook Pro display. Use when commands must be typed into or watched in the user's real Terminal.
---

# Visible Terminal control

## Space

- Host: MacBook Pro
- App: Terminal
- Target display: `MacBook`
- Window controls: Rectangle-derived `WinMove` and `WinSnap` inside Macro Recorder

## Procedure

1. Open Terminal and place it without guessing coordinates:

   ```ahk
   BlockInput "Off"
   Run "/usr/bin/open -a Terminal"
   Sleep 1200
   WinActivate "Terminal"
   WinRestore "Terminal"
   WinSnap "Terminal", "almost-maximize", "MacBook"
   WinActivate "Terminal"
   ```

2. Verify with:

   ```bash
   curl -fsS http://127.0.0.1:8829/status
   ```

   `front_now` must be `Terminal` when focus is intended.

3. To type a command into the visible Terminal:

   ```ahk
   BlockInput "Off"
   WinActivate "Terminal"
   SendText "command here"
   Send "{Enter}"
   ```

4. Prefer shell tools for commands that do not need visible typing. Use Macro Recorder for commands that must be seen or
   entered in the Terminal UI.

## Verification

- Confirm Terminal is frontmost after the last action.
- Read command output rather than treating the Enter key as success.
- Close/restore only the exact Terminal window that was opened.

## Turning point

- 2026-11-26: `WinSnap "Terminal", "almost-maximize", "MacBook"` reliably moved Terminal without coordinate guesses.
- 2026-11-26: Macro Recorder's guard restored focus to VS Code until `BlockInput "Off"` was sent before the final
  `WinActivate`.
