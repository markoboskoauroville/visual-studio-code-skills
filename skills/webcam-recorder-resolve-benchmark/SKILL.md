---
name: webcam-recorder-resolve-benchmark
description: Generate a known-delay waveform benchmark and compare Webcam Recorder sample-accurate AUTO ALIGN with DaVinci Resolve AutoSyncAudio. Use whenever changing sync analysis or after any reported precision mismatch.
---

# Webcam Recorder versus Resolve waveform sync

## Space

- Webcam Recorder: `~/Developer/WEB_CAM_VIEWER`
- Resolve Actions: `~/Developer/DAVINCI_RESOLVE_ACTIONS`
- Resolve router: `resolve-run`
- Test: 6,578 samples at 48 kHz = 137.041667 ms

## Reproduce

```bash
cd ~/Developer/WEB_CAM_VIEWER
python3 tools/make_sync_benchmark.py
"/Applications/Webcam Recorder (Mantra).app/Contents/MacOS/WebCamViewer" \
  --audio-file-selftest /tmp/wcr-benchmark-combined.mkv
resolve-run hold copilot-ltc-benchmark 3
resolve-run --wait 30 --timeout 120 --who copilot-ltc-benchmark \
  "$HOME/Developer/DAVINCI_RESOLVE_ACTIONS/scripts/Benchmark Audio Sync.py" \
  /tmp/wcr-benchmark-camera.mov /tmp/wcr-benchmark-selected.wav
resolve-run release copilot-ltc-benchmark
```

## Acceptance rule

- Webcam Recorder must report `-137.042 ms` with zero/near-zero window spread.
- Resolve must report an `Audio Offset` in the same direction and magnitude. At 25 fps, Resolve's 3.50 frames means
  140.000 ms on the reference signal.
- Difference must be no more than half a frame (20 ms at 25 fps). Webcam Recorder keeps sample precision, so a
  2.958 ms difference from Resolve's half-frame readout is a pass and is more precise.
- Read `Synced Audio` and the `Audio Offset` from Resolve, not only `AutoSyncAudio=true`.

## Safety

- Generate benchmark files under `/tmp`; they are disposable and must never replace a user's clip.
- Use the Resolve router, never a second direct `DaVinciResolveScript` connection.
- Place all Webcam Recorder window tests on the MacBook Pro display:

  ```ahk
  BlockInput "Off"
  WinSnap "Webcam Recorder (Mantra)", "almost-maximize", "MacBook"
  WinActivate "Webcam Recorder (Mantra)"
  ```

## History

- 2026-11-26: Webcam Recorder measured -137.042 ms across three windows. Resolve measured +3.50 frames = 140 ms.
  Difference 2.958 ms is inside half a 25 fps frame, while Webcam Recorder retained sample precision.
