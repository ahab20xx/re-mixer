# re-mixer — handoff notes

A single-file web app for playing live audio and video effects over a song,
styled like Windows 98. Runs on desktop Chrome and Android Chrome.

- Working copy: `C:\Users\sonic\Desktop\re-mixer\index.html`
- Repo: https://github.com/ahab20xx/re-mixer
- Live: https://ahab20xx.github.io/re-mixer/ — GitHub Pages, deployed from `main`

Everything is in `index.html` (~490 KB). There is no build step: open the file,
reload to see changes. Hard-reload (Ctrl+F5) after editing — the page caches
hard, and that has cost real time on this project.

---

## Read this first

The owner is not a programmer. Explain in plain language, say what you
actually changed, and do not describe anything as working until you have run
it. Several real problems in this project came from an assistant writing code
against assumed internals and then reporting success. See **Working on this
safely** at the bottom; it is the most important section here.

### Owner's standing preferences

- Suggest ways to save tokens when possible.
- If a task is easy for them but token-heavy for the assistant, say so plainly
  in capitals before doing it.
- Offer options before uploading visual changes such as logos.

---

## Dependencies (all of them)

- **98.css** 0.1.21 from jsDelivr, unpkg as fallback — the Windows 98 styling
- **Google Fonts** — the pixel typeface
- **lamejs** 1.2.1, loaded lazily the first time you export MP3

Nothing else. WAV, FLAC and AIFF are written byte by byte in this file. Beat
detection, key detection and the melody/harmony analysis are hand-rolled.
There is no audio-analysis library and no MIDI support. An earlier version of
this file credited Meyda and claimed MIDI; both were fabrications.

---

## How the code is laid out

**Audio** runs through a single `AudioWorkletProcessor` whose source sits in
this same file. It owns `pos` (the read head), `ghost` (the true timeline),
looping, stutter, reverse, tape stop and the chaos-reel shuttle. The main
thread talks to it with `send({t:'...'})`. Message types currently handled:
`load play pause seek region grid redetect xy xyp pitch stut stdir shuttle
tape rev pump gate crush ring smear stp half double close`.

**Video** is an HTML `<video>` kept in sync with the audio position, drawn to
canvas each frame: `drawVideoFx` (warble, aqua sway, etc.) → `drawScene`
(Aqua / Acid / Angel) → the CRT pass (`crtBegin`, `crtWarp`, `crtEnd`).

**Modules** are entries in `MODS`, arranged by the user on a grid:

| id | name |
|---|---|
| `xy` | FX PAD A |
| `xy2` | FX PAD B |
| `pd` | PUMP · DUCK |
| `rt` | REVERSE · TAPE |
| `stut` | STUTTER |
| `pitch` | PITCH |
| `aquaM` | AQUA |
| `acidM` | ACID 303 |
| `angelM` | ANGEL VOICES |
| `gate` | GATER |
| `dnb` | PS DRUM & BASS |

**Settings** come from three tables: `SPEC` (per module), `FXSPEC` (per
effect) and `screenRows()` (per screen). `panelRows(id)` assembles them. Rows
marked `adv:1` hide behind a "More settings" fold.

**Values** go through `getV(ns,k)` / `setV(ns,k,v)`, backed by
`localStorage['remixer.cfg']`. The full set of keys the app writes:
`remixer.cfg`, `remixer.layout3`, `remixer.layout2` (legacy, migrated),
`remixer.v`, `remixer.font`, `remixer.viewer`, `remixer.mix`, `remixer.vlock`,
`remixer.vpan`, `remixer.recfmt`, `remixer.chLive`, `remixer.more.*`,
`remixer.dnbAdded` (one-time flag: puts the drum & bass module on an old
saved layout once; removing the module afterwards keeps it removed).

**Chaos toys** (`chaosDrive`) spin a 3D object to scramble a module's
settings. The tape one is different: it shuttles the transport instead.

---

## Traps that have already bitten

- **`defOf` resolution order.** It checks `PADDEF`, then `SCREENS`, then the
  spec tables. A new screen setting whose default is not added to
  `SCREENDEF`/`SCREENS` renders as `undefined%`.
- **`<select>` returns strings.** A numeric dropdown option carrying the
  number `3` never matches a stored `"3"`, so the control silently snaps back
  to its first entry. The handler now coerces numerics; keep it that way.
- **Brace-less loops.** Appending a statement to `for(…) doThing();` puts it
  outside the loop. This shipped a `ReferenceError` that only fired when a
  fish jumped.
- **Per-key scene state.** `scn[key]` holds separate bubbles and particles for
  each screen, so two screens never render identically. Any test that
  compares one screen against another will see a few hundred pixels of
  difference that are nothing to do with the change.
- **The animation loop clears per-frame arrays.** Reading something like
  `FISHLIT` in a separate call from the render that filled it returns zero.
  Measure inside the same expression.
- **Stale files served from Pages.** A leftover `Re-mixer.html` sat in the
  repo for a day serving a frozen copy of the app to anything bookmarked to
  it. Deleted. `index.html` is the only page the site serves.

---

## Build stamp and the reset dialog

`const BUILD='…'` near `const wcan` is bumped on every save. It shows under
the "Load song…" prompt and in the **?** dialog on the menu bar. **Bump it
whenever you change the file** — it is how the owner tells whether a device is
running the current copy, and it settled a long-running "why is my phone not
updating" question in one glance.

The ? dialog also has:

- **Reload fresh copy** — clears the Cache API, unregisters service workers,
  reloads with a cache-busting query. Settings kept.
- **Clear saved data** — the same, plus removes every `remixer.*` key.

---

## Testing

There is no test suite in the repo. Changes are checked with a throwaway
Playwright + headless Chrome harness:

1. Copy `index.html`, injecting `window.__G=c=>eval(c);` before
   `function padAmts(id){` so a test can reach inside the main closure.
2. Load over `file://`, seed `localStorage` with a layout, reload.
3. Drive it with `__G("…")`, assert on state, screenshot a module.

Two limits worth knowing:

- **No audio device in headless Chrome.** The worklet never starts, so
  anything downstream of `ctx` cannot be exercised — only message plumbing and
  pure logic. Sound changes are structural checks, never heard.
- **Syntax-check before screenshotting.** Extract the `<script>` body and run
  `node --check`. A page with a syntax error still draws a window frame and
  can look fine.

---

## Working on this safely

Every one of these comes from something that actually went wrong here.

1. **Read the real code before patching.** Do not write edits against what you
   assume the internals look like.
2. **Make patches assert that each target string exists, and check the patch
   reported success.** A failed patch writes nothing and the app still loads,
   so the next test passes and proves nothing. A radiation feature was
   described to the owner as finished, and committed, when the patch had
   aborted and written zero lines.
3. **Test the patched copy**, not the file you started with.
4. **Commit messages must describe the diff.** Two commits here were wrong
   about their own contents — see below.
5. **Verify before reporting.** Measure, screenshot, and say what the
   measurement was. If something cannot be verified from here — audio, or the
   live site — say so rather than implying it was checked.

### Known-bad commit messages

- `137723c` "feat: add Radiation FX option to Old TV filter" contains **no
  radiation code**. It actually holds the settings-menu reorganisation, the
  Angel swell toggle and the Aqua bubble voices. Radiation landed later, in
  the commit that also rewrote this file.
- `f5eff0f` describes ripple and vowel work but also contains a full rewrite
  of the fish plop sounds.

Read the diffs, not the subject lines.

---

## Recent work, for context

Newest first. All of this is in `main`.

- Drum & bass chops: hold a break and slide up (CHOP edit, 1/8 repeat, 1/16
  and 1/32 stutter with rising pitch) or down (half-time, reverse, 3-step
  triplet loop); slide sideways in the same row to swap breaks mid-chop.
  Sliding on SNARE/HATS shifts roll speed. Chop changes land on the next
  step (`dn.chopPend`). Setting `cdist` = pixels per chop level.
- PS DRUM & BASS module (`dnb`). Four two-bar breaks
  (`DNB_PAT`), snare and hat rolls, a Reese bass and a sub drop, all
  synthesized in the file, rendered at 22 kHz / 10-bit for the PS1 feel.
  Scheduled from `dnbSched` (25 ms timer) on the song's *ghost* timeline, so
  stutter and reverse don't drag the drums; with the song paused it free-runs
  at the song's tempo. Beat rate Auto picks ½×/1×/2× nearest 172 BPM. Setting
  values are text (`x2`, `r16`) on purpose: the chip handler turns
  number-looking strings into numbers. Timing checked by rendering through an
  OfflineAudioContext with a proxied clock: hits within 0.0006 of a 16th.
  Not heard on a real device yet.
- Reverse and Tape Stop merged into one `rt` module; old layouts migrate
  automatically and the separate modules are retired.
- Module menu bars and the chaos readout locked to one line; overflow scrolls
  sideways instead of stealing height from the canvas.
- Acid 303's Accent and Slide pads hidden by default, behind a **Perform
  pads** setting.
- Fish: skeletons in the viewer at Radiation Hot and above; a green glow when
  acid is running, with the acid backdrop dimmed behind each fish so they read
  against it.
- Radiation screen effect — animated grain, monochrome, static tears, rolling
  tracking band, inverted frames. Viewer menu and a per-screen slider.
- Waveform scrollbar for panning while zoomed.
- Pitch module: eight selectable pad sets instead of one fixed row.
- Stutter: Direction (forward / reverse / ping-pong) replaced the old Slide
  mode chooser.
- Chaos reel winds the song; with Spin-up on it spins back up on release.
- Chaos toys: dragging up now tilts up.
- Aqua: musical bubble bloop, six fish plop voices with a picker, ripples
  confined to below the waterline.
- Angel: Volume swell mode picks its own vowel, OOH through AAH.
