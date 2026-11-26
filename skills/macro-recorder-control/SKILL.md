---
name: macro-recorder-control
description: Observe and control the Mac through the installed Macro Recorder (Mantra) API, including safe scripts, visual triggers, queued input jobs, and verification. Use for Mac UI actions when native Computer Use is unavailable or undesired.
---

# Macro Recorder control

## Space

- App: `/Applications/Macro Recorder (Mantra).app`
- CLI: `~/.local/bin/macro`
- Loopback API: `http://127.0.0.1:8829`
- Copilot extension: `mantra-macro-control`
- Tools: `mantra_macro_status` and `mantra_macro_run`

## Procedure

1. Read status before acting:

   ```bash
   curl -fsS http://127.0.0.1:8829/status
   ```

2. Verify `app=macrorecorder`, the intended version, `lock=off` or the expected owner, and no conflicting run.
3. Run a small script:

   ```bash
   "$HOME/.local/bin/macro" do - <<'AHK'
   WinActivate "Visual Studio Code"
   Sleep 100
   AHK
   ```

4. If the action must leave another app in front, start with:

   ```ahk
   BlockInput "Off"
   ```

   and activate the intended app last. Otherwise Macro Recorder restores the original foreground place.

5. For a script that is supposed to continue unattended, prefer a **Visual Trigger** over a long `macro do` process.
   The first picker captures the button/image; the second captures the restricted search area.

## Queue and dashboard

Macro Recorder v35 exposes:

- `GET /runs`
- `POST /run {"name":"label","script":"..."}`
- `POST /stop {"id":123}` or no id for all
- `GET /visual-triggers`
- `POST /visual-trigger {"macro":"...","image":"...","on":true,"zone":{"x":0,"y":0,"w":100,"h":100}}`

Scripts that use keyboard or mouse are FIFO-queued. Background jobs may overlap. Never assume that a second input
action will start immediately; read `/runs` and check `state=queued` versus `state=running`.

## Visual Trigger

1. Select the macro in the **MACRO** picker.
2. Press **VISUAL TRIGGER**.
3. Drag over the button/image.
4. Drag the allowed search area. Press Escape only when every screen should be searched.
5. Arm it with **EYE ON**.
6. Read `/visual-triggers` after setup.

## Safety

- Hold the input only for the shortest burst. The lock is a lease and Marko can take over.
- Exit/stop on `exit 130`, "ABORTED", or TAKE OVER; do not retry the same UI action blindly.
- Accessibility first, OCR/image matching second. Read back state after toggles.
- Never use Macro Recorder's window controls to bypass a system security prompt.

## Turning points

- 2026-11-26: Native Computer Use failed with `client is not authorized to use this service` even though macOS
  Accessibility and Screen Recording were enabled. A user-scoped Copilot extension calling Macro Recorder restored
  verified control.
- 2026-11-26: A running macro restores the original front app unless `BlockInput "Off"` is deliberately used and the
  target is activated last.
- 2026-11-26: A queued run was counted as an active input run, preventing it from advancing. The fixed rule counts only
  `usesInput && state=running` when deciding whether the FIFO head can start.
