---
name: voice-to-video
description: Turn a voice recording into a published video. Silence-cut, transcribe, render, QA, schedule.
version: 1.0.0
license: MIT
---

# Voice to Video

Takes a raw voice memo and turns it into a finished, captioned video scheduled to a channel, with a review gate before anything goes live. Built and validated on a real publishing pipeline: more than 60 rendered videos, one batch of 10 QA-passed renders, several daily automated slots.

## When to use

- You have a voice recording (a memo, a dictated monologue, a podcast take) and want it to become a video.
- You publish on a schedule (daily or weekly slots) and want the scheduling handled.
- You want captions cut from real speech, with the noise cases handled, not assumed away.

## What you need

- ffmpeg on the render machine.
- A local speech-to-text engine. We use whisper on Apple Silicon (MLX builds); any offline whisper works.
- Background footage, or still images for the Ken Burns mode.
- A scheduling API for your platform. We used a social scheduling service with a presigned S3 upload; any service with an upload + "schedule a post" endpoint fits.

## Pipeline

1. **Cut the silences.** ffmpeg `silenceremove`, threshold -38 dB. Three pacing presets: dynamic (>0.45 s cut, 0.18 s kept), paced (0.55/0.25), calm (0.7/0.35). Pick per topic. The rhythm is the video: cutting pauses changes the feel more than any visual.
2. **Transcribe locally.** Full model path if your tool needs it (bare "small" can fail to resolve). Do not chunk long files by hand: windowed transcription handles a 15-minute take in one call, and hand-rolled byte splits break the decoder.
3. **Render.** Two modes, both validated:
   - **Wide (16:9):** footage full-frame, no burned captions (platform auto-captions stay readable), ambience track 12% under the voice.
   - **Vertical (9:16):** crop, discreet burned captions (style below).
   - **Stills mode (Ken Burns):** the classic failure is `zoompan` with a long `d=` fed by a single image ("Could not open encoder before EOF"). Working recipe: encode one segment per image with `-framerate 30 -loop 1 -t <duration>` and `zoompan=...:d=1` (one output frame per input frame, zoom accumulates), then concat, then a final pass for captions + audio. Keep `-framerate 30` BEFORE the `-loop 1 -t` input options or a 10 s segment renders 8.33 s.
4. **Clean the footage.** Crop the bottom 12% of stock footage: overlays and watermarks live there. Keep the original file as a second input: the ambience audio comes from it.
5. **QA before anything ships.** Extract a frame at 30 s, plus a 2x zoomed crop of the caption band for verticals. Check: no watermark, captions match the style spec, the face of the video is right. Tiling several frames with a glob feeds them alphabetically: name files `01_start`, `02_mid`, `03_end` or the reviewer reads your video backwards.
6. **Publish through a review gate.** Every render waits as "ready, needs approval". Nothing is scheduled without an explicit go from the owner. Check the calendar of scheduled posts first: colliding slots are the most common scheduling bug.

## Caption style (validated: "you should not see the captions")

Arial 40 px, white at alpha 0x60 (semi-transparent), no outline, no shadow, bottom-center, generous bottom margin so the platform UI never covers them, max 2 lines of 30 characters, no overlapping cue times. Judge the style on a 2x zoomed crop of the caption band: on a full-frame screenshot, 40 px text looks deceptively huge.

## Editorial gates (before any render)

1. **Find the key moment.** A raw memo is mostly banal. Extract the strongest 30-60 s passage with context, instead of keeping the mass.
2. **Privacy, non-negotiable.** Passwords, credentials, sensitive personal details: the whole recording is out. Third parties identifiable by name or private life: cut those moments. In doubt, cut. Protecting people in the recording matters more than the story.
3. **Spheres.** Classify before rendering: public ideas (publishable), work content (never on a personal channel), private life (never published). When in doubt: private. Never mix a work story into a persona video.
4. **Noise detection.** Restaurant or street noise makes small models hallucinate: the signature is a short phrase repeated in a loop for minutes. Retry with a larger model; if it is still bad, mark the take "too noisy" and skip. Never burn captions from a hallucinated transcript.
5. **One thing at a time on the render machine.** If the owner uses the machine, serialize the renders (one ffmpeg at a time, `nice -n 19`, check the load before each render, stop if it climbs) or run the batch at night. A batch that makes the machine unusable is a failure even if every render succeeds.

## Fixing caption typos without re-transcribing

The naive fix (edit the transcript, re-run the pipeline) does not work: the pipeline re-transcribes and re-cuts on its own intermediate file, so your corrections are silently ignored. Working sequence: QA frames first, list the typos; edit the caption file directly (the timings stay valid); re-render only the final pass, passing the existing cut audio and the corrected caption file; re-QA one frame. A 2-minute re-render instead of a full re-transcription.

## Scheduling notes

- Vertical videos are capped (60 to 180 s depending on platform): check the duration gate before promising a slot.
- Large uploads can hit a size cap on presigned single-part upload: recompress long videos before the upload, not after a hard failure.
- Editing a scheduled post through an update endpoint can create a duplicate. If your scheduler behaves this way, edit = delete + recreate.
- Titles carry the video: subject-linked and attractive, never the recording date, never the filename. Working titles are internal placeholders only.

## Verification checklist

- [ ] The rendered file plays, correct aspect ratio, voice clear over ambience.
- [ ] One frame QA passed (plus caption-band crop for verticals).
- [ ] Duration fits the slot type.
- [ ] The transcript contains no hallucination loop.
- [ ] The schedule shows the post at the right slot, no collision.
- [ ] The tracking table (whatever you use) shows the new row with the approval state.

## Pitfalls

- Two `-filter_complex` in one ffmpeg command: the second overwrites the first (you get two audio tracks). Put video and audio mixing in ONE filter graph.
- `str.replace` patches fail silently on no-match: verify each patched line with a grep.
- A killed batch has usually already delivered part of its work: check the state file and the rendered-files list before relaunching, and relaunch only the remainder.
- Judge batch progress on the state file growing, never on the log or the exit code: a pipeline ending in `| tee` reports success even when it was killed.
- Record times in voice archives: use the embedded `creation_time` tag, never the file mtime (the mtime is the export date).
- If your agent cannot see images in the current session, delegate frame QA to a tool that can, and never accept a caption verdict from a session that answers "I cannot see the image".
