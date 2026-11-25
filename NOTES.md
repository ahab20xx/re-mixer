# re-mixer — notes for whoever picks this up next

A single-file web app for playing live audio and video effects over a song.
Windows 98 look. Runs on desktop Chrome and Android Chrome.

- Working copy: `C:\Users\sonic\Desktop\re-mixer\index.html`
- Repo: https://github.com/ahab20xx/re-mixer
- Live: https://ahab20xx.github.io/re-mixer/ (GitHub Pages, from `main`)

Everything is in `index.html`. There is no build step. Open the file, reload to
see changes. Hard-reload (Ctrl+F5) after editing — the page caches aggressively.

## Correction to the git log

Commit `137723c` is titled "feat: add Radiation FX option to Old TV filter" but
contains **no radiation code at all**. The patch that was supposed to add it
failed silently and the commit went ahead with the wrong message. What that
commit actually holds is the settings-menu reorganisation, the Angel swell
toggle and the Aqua bubble voices. Radiation was added later, separately.
Commit `f5eff0f` is also under-described: besides the ripple and vowel work in
its message, it contains a full rewrite of the fish plop sounds.

Read the diffs, not the subject lines.

## What is actually in here

External dependencies (all of them):

- `98.css` 0.1.21 from jsDelivr, with unpkg as fallback — the Windows 98 styling
- Google Fonts — the pixel typeface
- `lamejs` 1.2.1, loaded lazily from jsDelivr/unpkg the first time you export
  MP3. Every other export format is encoded by hand in this file (WAV, FLAC,
  AIFF are all written out byte by byte).

There is no audio-analysis library. Beat detection, key detection and the
melody/harmony analysis are all hand-rolled in this file. There is no MIDI
support.

## Shape of the code

- **Audio** runs through one `AudioWorkletProcessor` whose source lives in this
  same file. It owns playback position (`pos` for the read head, `ghost` for the
  true timeline), looping, stutter, reverse, tape stop and the chaos-reel
  shuttle. The main thread talks to it with `send({t:'...'})` messages.
- **Video** is an HTML `<video>` kept in sync with the audio position, drawn to
  canvas every frame with the effects layered on top (`drawVideoFx`), then the
  Aqua/Acid/Angel scenes (`drawScene`), then the CRT pass.
- **Modules** are entries in `MODS`: FX Pad A/B, Pump·Duck, Stutter, Pitch,
  Tape Stop, Reverse, Aqua, Acid 303, Angel Voices, Gater. The user arranges
  them on a grid; the layout lives in `localStorage` under `remixer.layout3`.
- **Settings** come from three tables: `SPEC` (per module), `FXSPEC` (per
  effect) and `screenRows()` (per screen). `panelRows(id)` assembles them.
  Rows marked `adv:1` hide behind a "More settings" fold.
- **Values** go through `getV(ns,k)` / `setV(ns,k,v)`, backed by
  `localStorage['remixer.cfg']`. Defaults resolve via `defOf`, which checks
  `PADDEF`, then `SCREENS`, then the spec tables — **a new screen setting needs
  its default added to `SCREENDEF`/`SCREENS` or the slider reads `undefined`**.
- **Chaos toys** (`chaosDrive`) spin a 3D object to scramble a module's
  settings; the tape one shuttles the transport instead.

## Testing changes

There is no test suite in the repo. Changes have been checked with a throwaway
Playwright + headless Chrome harness:

1. Copy `index.html`, injecting `window.__G=c=>eval(c);` before
   `function padAmts(id){` so a test can reach inside the main closure.
2. Load it over `file://`, seed `localStorage` with a layout, reload.
3. Drive it with `__G("...")` and assert on state, or screenshot a module.

Two things worth knowing: there is no audio device in headless Chrome, so the
worklet never starts and anything downstream of `ctx` can't be exercised that
way — only the message plumbing and pure logic. And syntax-check before
screenshotting, by extracting the `<script>` body and running `node --check`;
a page with a syntax error still renders a window frame and can look fine.

## Working with another assistant

Point it at this file first. Useful ground rules, learned the hard way:

- Read the real code before patching. Several bugs here came from edits written
  against assumed code that didn't exist.
- Make patches assert that each target string is present, and check the patch
  reported success — a failed patch writes nothing and the app still loads.
- Test the **patched** copy. A test run against the old file passes happily and
  proves nothing.
- Commit messages should describe the diff. See the correction above.

To work from the repo: clone it, edit `index.html`, commit and push. Pages
redeploys from `main` on its own.
