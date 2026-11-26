---
name: pixel-ltc-safety
description: Start, verify, measure, and immediately stop the annoying Pixel LTC test sound, including safe ADB volume control and silent restoration. Use only around an actual LTC recording/test.
---

# Pixel LTC: loud only for the minimum required measurement

Marko warned that LTC audio is extremely annoying to hear: turn it on only while recording/testing and turn it off as
soon as the take or experiment ends.

## Space

- Pixel serial: `41241FDH2S10SJ`
- App: **TC LTC JAM**
- Package: `com.mantraproductions.tcltcjam`
- Activity: `.MainActivity`
- v2+ ADB control: boolean intent extra `ltc`
- Recording reference: Webcam Recorder's selected microphone plus the phone's visible 25 fps timecode

## Required start sequence

1. Confirm the Pixel app is foreground:

   ```bash
   adb -s 41241FDH2S10SJ shell dumpsys activity activities \
     | rg -m1 'com\.mantraproductions\.tcltcjam/.MainActivity'
   ```

2. Save the current music volume state before changing it:

   ```bash
   adb -s 41241FDH2S10SJ shell dumpsys audio \
     | sed -n '/- STREAM_MUSIC:/,+9p'
   ```

3. Raise volume only if the recording cannot lock cleanly. Read the phone's `STREAM_MUSIC` Max and use ADB volume keys;
   never infer the current setting from the UI:

   ```bash
   adb -s 41241FDH2S10SJ shell input keyevent KEYCODE_VOLUME_UP
   ```

4. Start LTC deterministically:

   ```bash
   adb -s 41241FDH2S10SJ shell am start -W \
     -a android.intent.action.MAIN \
     -n com.mantraproductions.tcltcjam/.MainActivity \
     --ez ltc true
   ```

5. Verify a screenshot says `AUDIO LTC OUT: ON`:

   ```bash
   adb -s 41241FDH2S10SJ exec-out screencap -p > /tmp/tc-ltc.png
   ```

6. Record only the necessary interval.

## Mandatory stop sequence

Stop LTC immediately after the take; do not leave it running between steps:

```bash
adb -s 41241FDH2S10SJ shell am force-stop com.mantraproductions.tcltcjam
adb -s 41241FDH2S10SJ shell am start -W \
  -a android.intent.action.MAIN \
  -n com.mantraproductions.tcltcjam/.MainActivity
adb -s 41241FDH2S10SJ exec-out screencap -p > /tmp/tc-ltc-stopped.png
```

The final screenshot must say `AUDIO LTC OUT: OFF`.

Restore media volume if it was changed, using the recorded `streamVolume` and `KEYCODE_VOLUME_DOWN`/`KEYCODE_VOLUME_UP`,
unless Marko explicitly chooses a new level. Never leave sound enabled to avoid changing it later.

## Verification

- A clean LTC file must pass both the decoder/matched-filter and multiple OCR frames.
- Webcam Recorder must refuse to apply a delay when frame matches disagree; a false number is worse than no result.
- The real-device failure message must name a next action such as moving the phone closer or raising its speaker volume.

## Turning points

- 2026-11-26: direct ADB taps in rotated landscape hit the phone's volume control and lowered `STREAM_MUSIC` from
  25 to 1. Do not use guessed touchscreen coordinates for LTC.
- 2026-11-26: TC LTC JAM v2 added an ADB intent because coordinate tapping was unreliable and unsafe.
- 2026-11-26: LTC was restored to `25`, one test take was recorded, then the package was force-stopped and reopened
  without the extra. The final screenshot confirmed `AUDIO LTC OUT: OFF`.
