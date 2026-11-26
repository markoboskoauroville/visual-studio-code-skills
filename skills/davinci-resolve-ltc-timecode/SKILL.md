---
name: davinci-resolve-ltc-timecode
description: Decode LTC from a recording, create a Resolve-compatible copy, import it through the one Resolve router, and set/read back the media clip Start TC. Use for LTC/video clips opened in DaVinci Resolve.
---

# Resolve clip timecode from LTC

## Space

- Source recorder: Webcam Recorder MKV, two audio tracks possible
- Resolve repository: `~/Developer/DAVINCI_RESOLVE_ACTIONS`
- Router CLI: `~/.local/bin/resolve-run`
- Router repository module: `scripts/resolve_router.py`
- Import script: `scripts/Import LTC Timecode.py`
- Official local typings:
  `/Library/Application Support/Blackmagic Design/DaVinci Resolve/Developer/Scripting/DaVinciResolveScript.pyi`

## Procedure

1. Never connect directly with `DaVinciResolveScript.scriptapp()`. Check the one router first:

   ```bash
   resolve-run status
   resolve-run --wait 30 --who <job-name> -c 'result = project.GetName() if project else None'
   ```

2. Extract the selected audio track to PCM:

   ```bash
   ffmpeg -nostdin -hide_banner -loglevel error -y \
     -i /path/take.mkv -map 0:a:0 -ac 1 -ar 48000 -c:a pcm_s24le /tmp/take-ltc.wav
   ```

3. Decode real LTC with libltc:

   ```bash
   ltcdump -f 25 /tmp/take-ltc.wav
   ```

   Require valid sequential frames. Nonsense hours/minutes/seconds, one discontinuity, or no frames means the audio
   path was clipped/distorted; do not write a fake Start TC.

4. Resolve may not import the recorder's MKV directly. Make a new MOV without re-encoding:

   ```bash
   ffmpeg -nostdin -hide_banner -loglevel error -y \
     -i /path/take.mkv -map 0 -c copy -movflags +faststart /path/take_resolve.mov
   ```

   Never overwrite an original or a media file used by Resolve.

5. Determine the clip's frame-zero timecode from the first valid LTC frame:

   - decode its `timecode` and audio sample position;
   - subtract the sample position at that frame from the code, rounded to the clip's frame rate;
   - for a non-decodable physical LTC path, the visually OCR'd phone timecode is only a labelled fallback because
     TC LTC JAM's display and LTC share the same generator.

6. Import and write the source property through the router:

   ```bash
   resolve-run --wait 30 --timeout 60 --who copilot-ltc \
     "/Users/markobosko/Developer/DAVINCI_RESOLVE_ACTIONS/scripts/Import LTC Timecode.py" \
     /path/take_resolve.mov 04:11:50:20
   ```

7. Require the result to show `startTCAfter` equal to the requested code and `selected=true`.

## Official property

Blackmagic's installed Resolve API typing defines:

```python
item.GetClipProperty("Start TC")
item.SetClipProperty("Start TC", "HH:MM:SS:FF")
```

Always read it back; `SetClipProperty` returning true alone is not the complete verification.

## Safety

- Keep only one Resolve API connection by using `resolve-run`.
- A busy/timeout response means wait; never open a second direct `DaVinciResolveScript` connection.
- Do not append the clip automatically unless the user asked for a timeline edit.
- Never overwrite source media.

## History

- 2026-11-26: Marko requested this workflow. The existing resolver repository was reused, `ltc-tools` supplied a real
  decoder, and a lossless MOV import plus `Start TC` readback script was added.
- 2026-11-26: The synthetic LTC encoder decoded exactly with `ltcdump`, but the room/speaker camera take produced invalid
  values. Treat distorted physical LTC as untrusted; use a direct audio connection or improve the speaker/mic path.
