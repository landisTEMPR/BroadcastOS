# BroadcastOS

A control surface for vMix. Edit titles, build a run of show, drive audio, and
wire scenes together — without leaving the app or nursing a spreadsheet.

Built for live production: nothing it does should surprise you mid-show.

---

## Install

1. Download the latest `BroadcastOS-x.y.z-setup.exe` from
   [Releases](../../releases).
2. Run it. No admin rights needed — it installs per user by default.
3. In vMix, turn on **Settings → Web Controller**. BroadcastOS talks to vMix
   over HTTP on port 8088 and the TCP event stream on 8099.
4. Launch BroadcastOS, leave the host as `127.0.0.1` if vMix is on the same
   machine (or enter its IP), and press **Connect**.

Windows 10 or 11, 64-bit. Everything else it needs ships with the installer.

Windows will warn that the publisher is unrecognised, because the installer
isn't code-signed yet. Choose **More info → Run anyway**.

### Updates

BroadcastOS checks for a new version a few seconds after it starts and offers
to install it. Accepting closes the app, updates, and reopens it. Your shows,
triggers, hotkeys and settings are kept.

You can also check on demand from **File → Check for updates**.

---

## What it does

### Titles

Every Title and GT input, with its editable text and image fields. Image
layers are detected however vMix reports them — `.Source` names, `<image>`
elements, or a value that's simply a picture path — and get a file browser.

Editing is safe while the show is running. vMix pushes state changes
constantly, and a naive editor throws away whatever you're halfway through
typing. Here, a field you've touched is never overwritten: it's marked as
unpushed, and if vMix changes that same field underneath you it's flagged as a
conflict rather than silently clobbered.

- **Push** per field, or **Push all** to fire every edit at once so a
  half-typed lower third never hits air
- **Revert** to whatever vMix currently has
- **Undo last push** puts the previous value back
- Optional character budgets per field, with a live count that turns red — so
  a long name doesn't overflow the lower third on air

### Scenes and the graph

A scene in vMix is an input that composites others as layers. BroadcastOS
reads that directly from the preset, so scenes and their contents are
discovered rather than configured.

The **Graph** tab is a node editor in the spirit of a shader graph:

- Assets on the left, scenes on the right, bezier links showing what feeds what
- Drag a node's output dot onto a scene to add a layer, or onto a specific
  layer socket to replace it
- Drag across a link to cut it, which sets that layer to None
- Reorder layers, hide and show them, replace their source
- Expand any node to edit its fields in place
- Create, rename, duplicate and delete inputs
- Colour-coded by naming category (`GFX //`, `VFX //`, `FS //` …), with
  per-node and per-category overrides
- Pan, zoom, and a layout that's saved with your show

Layer edits are also available as buttons on the Titles tab if you'd rather
not work in the graph.

### Run of show

A rundown of **cues**. A cue is a named, ordered set of actions — set text,
set an image, take an overlay in or out, transition, cut, change a volume,
select a list item, wait, or fire any raw vMix function.

That one idea covers everything: a single GO can set three fields, duck the
music, and bring up the overlay in the right order.

- **+ Cue** on any title or scene builds a working cue from its current values
- Drag to reorder; **GO** fires and advances
- Auto-follow makes a cue fire the next one when it finishes
- Timing against planned durations, showing how far behind or ahead you are
- CSV import that watches the file, for anyone migrating off a spreadsheet
- Save and load the whole show as a `.vmixshow`

The **Timeline** tab is the same rundown drawn against time, with multiple
tracks so cues can overlap. Drag to move, drag the edge to stretch,
right-click for cue settings, double-click to recolour.

### Triggers

vMix's own triggers live in the input settings and aren't reachable through
its API, so BroadcastOS implements the same idea itself: watch for a state
change, run an attached action list. All five events are covered —
`OnTransitionIn`, `OnTransitionOut`, `OnOverlayIn`, `OnOverlayOut`,
`OnCompletion`.

The point of the setup is that you shouldn't have to type levels by hand. Set
the mixer how the scene should sound, press **Use the mixer as it sounds
now**, and every fader and mute becomes a fade action on that event.

Triggers stay disarmed until you tick **Arm triggers**, and the watcher seeds
its baseline on connect so joining mid-show doesn't fire everything at once.

### Audio

Master and busses A–G alongside every audio input. Faders, mute, solo, and
meters on a dBFS scale with green/amber/red zones, peak hold and a numeric
readout.

vMix reports volume on a different scale from the one its faders use — the
documented conversion is applied at the boundary, so a fader here matches the
one in vMix rather than bunching near the bottom.

Also: named mixer snapshots you can recall or attach to a scene, and a ducking
control that fades an input down and back.

### The rest

- **Lists** — every VideoList input as a tree, with transport and a countdown
  that turns red under ten seconds
- **Clock** — bind a title field to a wall-clock target and it rewrites every
  second, which is how a preshow countdown should work
- **Replay** — mark, play and transport controls, plus a list of clips
  exported this session
- **Show** panel — session stats, a usage report of what nothing touches, and
  your current hotkeys
- **Log** — every command sent, exportable as an as-run CSV for sponsor
  reconciliation
- **Operator mode** (F11) — fullscreen, stripped to current cue, next cue,
  tally, recording state and a very large GO
- **Phone entry** — serves a form on your network so someone else can type
  guest names from their phone while the operator stays on the switcher
- **OSC input** on UDP 9000 for `/vmix/go`, `/vmix/back`, `/vmix/cue/N`,
  `/vmix/panic`
- **Configurable hotkeys**, which also means Stream Deck support, since those
  send keystrokes

### Before you go live

- **Preflight** walks every cue for missing image files, renamed fields,
  inputs that have left the preset, and out-of-range list indices
- **Drift detection** compares a saved show against the preset vMix has
  loaded, and reports what was renamed, re-layered, added or removed
- **Snapshot and restore** captures every title's values and the overlay
  state, so rehearsal is free
- **Dry run** advances the rundown and fills the log while sending nothing to
  vMix

---

## Where your data lives

Shows, triggers, snapshots, hotkeys and panel layout are stored in:

```
%APPDATA%\BroadcastOS
```

Everything is written on quit, and an autosave runs a few seconds after any
change. Uninstalling leaves this folder alone.

---

## Known limits

Worth knowing before they surprise you.

**Layer edits on a live input.** vMix doesn't apply layer changes to an input
that's currently in Program. The command reports success and nothing happens.
Take the scene off air to re-wire it. BroadcastOS warns when you try.

**Trigger timing.** vMix's own triggers fire internally and instantly.
BroadcastOS detects state changes over HTTP, so its triggers land roughly
100–300 ms later. Fine for audio fades, overlays and follow-on takes; use
vMix's own triggers for anything frame-accurate.

**vMix categories.** The category tabs in vMix aren't exposed through its API,
so BroadcastOS groups inputs by the `CATEGORY // Name` prefix instead. Naming
an input `GFX // LOWER-THIRD` files it under GFX automatically.

**Ten layers.** That's vMix's limit, not one this app adds.

**Unsigned installer.** SmartScreen will warn until the project has a
code-signing certificate.

---

## Reporting a problem

The **Log** tab records every command sent to vMix and every reply. If
something misbehaves, that log usually shows which end went wrong — include it
when you open an issue, along with what you were doing and which version
you're on (shown under File → Check for updates).
