# Visualizer & desktop dancer

Brube2000 ships a music-reactive **visualizer**: a stylised, jointed dancer
(think Jamiroquai's hat) that grooves to whatever is playing, plus a spectrum
backdrop. It runs in three homes from the same code:

1. **In-app** — the **VIZ** button on the transport row opens a *Visualizer*
   window.
2. **On the desktop** — drag the dancer out of that window and it becomes a
   **replicant** living on the wallpaper (transparent background).
3. **Standalone** — double-click `Brube2000Dancer` for a small visualizer
   window on its own.

## Using it

- Press **VIZ** in the main window. The dancer appears and animates while music
  plays.
- To put it on the desktop: grab the small **drag handle** in the bottom-right
  corner of the dancer and drag it onto the wallpaper, then release. (Opening
  VIZ calls `BDragger::ShowAllDraggers()` so the handle is visible.)
- The desktop dancer reads the audio live, so it dances whenever Brube2000 is
  playing and idles (a gentle sway) otherwise. If you quit and relaunch
  Brube2000 it re-syncs on its own within ~1 second.
- Remove it like any replicant: right-click the handle → *Remove*.

## How it reacts to the music

The audio output taps every PCM buffer on the media thread into an
`AudioTap` (`src/audio/AudioTap.*`). The media thread does only cheap work —
downmix to mono, sum energy, publish a 1024-sample window under a seqlock — and
each reader does its own FFT. The tap exposes:

- **level** — smoothed loudness (drives the amplitude of everything),
- **beatCount** — an onset counter (energy spike vs. running average),
- the raw **window** — for a log-spaced spectrum.

The dancer (`src/gui/DancerView.h`) is not driven by a single sine (which reads
as a stiff scarecrow). It runs a small physics model: a free-running phase gives
the groove; hips and shoulders counter-rotate (contrapposto) with a weight
shift; beat onsets kick damped springs for the bounce (squash/stretch + hop),
an overhead-V arm throw ("wahoo"), and a hat wobble. Limbs are jointed bones
with a trailing forearm (follow-through) and ball joints, so the motion eases
and overshoots instead of snapping.

## Architecture notes (the non-obvious bits)

Two Haiku realities shaped the design:

- **Haiku has no window transparency.** `BWindow` has no opacity API and the
  app_server isn't a compositor, so a see-through visualizer window isn't
  possible. A **desktop replicant** is the way to get the dancer onto the
  wallpaper: it draws over the desktop, and where it doesn't draw, the wallpaper
  shows through.

- **A replicant is instantiated by `load_add_on()` on the owning binary.** If
  that binary is the whole app, its global constructors (protobuf descriptor
  registration) abort inside Tracker and the replicant is a grey zombie. So the
  replicant lives in a **tiny helper binary**, `Brube2000Dancer`, built from
  just the view + `AudioTap` and linking only the Interface Kit — no protobuf,
  no networking. The archived view points Tracker at the helper's signature
  (`application/x-vnd.brube2000-dancer`), never at the full app. The main app
  reuses the exact same `VisualizerView` for its in-app window.

Supporting details that all had to be right for the drop to work:

- `VisualizerView` is in the **global namespace** — Haiku resolves the replicant
  by demangling exported symbols and matching `VisualizerView::Instantiate`,
  which a namespace would defeat.
- `Instantiate` is defined **out-of-line** so the symbol is actually emitted
  into the binary's dynamic table (an unused inline static member is discarded).
- The helper binary is **stamped** (`rc`/`xres`/`mimeset`) with its signature so
  `be_roster->FindApp()` can resolve it, and it is shipped + registered by
  `make-hpkg.sh`.
- The in-app tap and the replicant share one **named `BArea`** so the reader in
  Tracker's process sees the same lock-free data. The reader re-clones the area
  when the app restarts (a new area takes the name), so a long-lived desktop
  replicant survives the app coming and going.

## Files

- `src/audio/AudioTap.{h,cpp}` — the shared-memory PCM tap (writer + reader).
- `src/gui/DancerView.h` — the replicable dancer view (shared by app + helper).
- `src/gui/DancerView.cpp` — out-of-line `Instantiate`.
- `src/gui/VisualizerWindow.{h,cpp}` — the in-app host window.
- `src/app/DancerApp.cpp` — the standalone/helper binary `Brube2000Dancer`.
- `Brube2000Dancer.rdef` — the helper's signature/version resources.
