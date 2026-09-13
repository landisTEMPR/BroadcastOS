# BroadcastOS

A control surface for vMix. Edit titles, build a run of show, drive audio, and wire scenes together without leaving the app.

## Install

1. Download the latest `BroadcastOS-x.y.z-setup.exe` from [Releases](../../releases).
2. Run it. It installs per user, so you don't need admin rights.
3. In vMix, turn on **Settings → Web Controller**. BroadcastOS uses HTTP on port 8088 and the TCP event stream on 8099.
4. Open BroadcastOS. Leave the host as `127.0.0.1` if vMix is on the same machine, or type its IP. Press **Connect**.

Windows 10 or 11, 64 bit. Everything else it needs ships with the installer.

Windows will say the publisher is unrecognised, because the installer isn't code signed yet. Choose **More info**, then **Run anyway**.

## Updates

BroadcastOS checks for a new version a few seconds after it starts, and offers to install it. Accepting closes the app, updates, and reopens it. Your shows, triggers, hotkeys and settings are kept.

You can also check any time from **File → Check for updates**.

## Titles

Every Title and GT input, with its editable text and image fields. Image layers are picked up however vMix reports them, whether that's a `.Source` name, an `<image>` element, or a value that's simply a path to a picture. Those fields get a file browser.

You can edit while the show is running. vMix pushes state changes constantly, and an editor that rebuilds itself on every one of them throws away whatever you're halfway through typing. Here a field you've touched is never overwritten. It's marked as unpushed, and if vMix changes that same field underneath you it's flagged as a conflict instead of being quietly replaced.

* **Push** one field, or **Push all** to send every edit at once, so a half typed lower third never hits air
* **Revert** to whatever vMix currently has
* **Undo last push** puts the previous value back
* Character budgets per field, with a live count that turns red, so a long name doesn't overflow the lower third on air

## Scenes and the graph

A scene in vMix is an input that composites others as layers. BroadcastOS reads that from the preset, so scenes and their contents are found rather than configured.

The **Graph** tab is a node editor:

* Assets on the left, scenes on the right, curved links showing what feeds what
* Drag a node's output dot onto a scene to add a layer, or onto a layer socket to replace it
* Drag across a link to cut it, which sets that layer to None
* Reorder layers, hide and show them, swap their source
* Expand any node to edit its fields in place
* Create, rename, duplicate and delete inputs
* Colours follow the naming category (`GFX //`, `VFX //`, `FS //`), and you can override per node or per category
* Pan, zoom, and a layout that saves with your show

The same layer controls sit on the Titles tab if you'd rather not work in the graph.

## Run of show

A rundown of **cues**. A cue is a named, ordered set of actions: set text, set an image, take an overlay in or out, transition, cut, change a volume, select a list item, wait, or fire any raw vMix function.

That covers everything in one idea. A single GO can set three fields, duck the music, and bring up the overlay in the right order.

* **+ Cue** on any title or scene builds a working cue from its current values
* Drag to reorder. **GO** fires and advances
* Auto follow makes a cue fire the next one when it finishes
* Timing against planned durations, showing how far behind or ahead you're running
* CSV import that watches the file, for anyone moving off a spreadsheet
* Save and load the whole show as a `.vmixshow`

The **Timeline** tab is the same rundown drawn against time, with several tracks so cues can overlap. Drag to move, drag the edge to stretch, right click for cue settings, double click to recolour.

## Triggers

vMix's own triggers live in the input settings and aren't reachable through its API, so BroadcastOS does the same job itself. It watches for a state change and runs an attached action list. All five events are covered: `OnTransitionIn`, `OnTransitionOut`, `OnOverlayIn`, `OnOverlayOut`, `OnCompletion`.

Setting them up shouldn't mean typing levels by hand. Get the mixer sounding right, press **Use the mixer as it sounds now**, and every fader and mute becomes a fade action on that event.

Triggers stay disarmed until you tick **Arm triggers**. The watcher takes a baseline when it connects, so joining mid show doesn't fire everything at once.

## Audio

Master and busses A through G alongside every audio input. Faders, mute, solo, and meters on a dBFS scale with green, amber and red zones, peak hold, and a numeric readout.

vMix reports volume on a different scale from the one its faders use. The documented conversion is applied at the boundary, so a fader here matches the one in vMix instead of bunching near the bottom.

There are also named mixer snapshots you can recall or attach to a scene, and a ducking control that fades an input down and back.

## Everything else

* **Lists**: every VideoList input as a tree, with transport and a countdown that turns red under ten seconds
* **Clock**: bind a title field to a wall clock target and it rewrites every second, which is how a preshow countdown should work
* **Replay**: mark, play and transport controls, plus a list of clips exported this session
* **Show**: session stats, a usage report of what nothing touches, and your current hotkeys
* **Log**: every command sent, exportable as an as run CSV for sponsor reconciliation
* **Operator mode** (F11): fullscreen, stripped back to current cue, next cue, tally, recording state and a very large GO
* **Phone entry**: serves a form on your network so someone else can type guest names from a phone while the operator stays on the switcher
* **OSC input** on UDP 9000 for `/vmix/go`, `/vmix/back`, `/vmix/cue/N` and `/vmix/panic`
* **Configurable hotkeys**, which also gives you Stream Deck support, since those send keystrokes

## Before you go live

* **Preflight** walks every cue looking for missing image files, renamed fields, inputs that have left the preset, and list indices past the end of a list
* **Drift detection** compares a saved show against the preset vMix has loaded, and reports what was renamed, re layered, added or removed
* **Snapshot and restore** captures every title's values and the overlay state, so rehearsal costs you nothing
* **Dry run** advances the rundown and fills the log while sending nothing to vMix

## Where your data lives

Shows, triggers, snapshots, hotkeys and panel layout are stored in:

```
%APPDATA%\BroadcastOS
```

Everything is written on quit, and an autosave runs a few seconds after any change. Uninstalling leaves this folder alone.

## Known limits

Worth knowing before they catch you out.

**Layer edits on a live input.** vMix won't apply layer changes to an input that's in Program. The command reports success and nothing happens. Take the scene off air to rewire it. BroadcastOS warns you when you try.

**Trigger timing.** vMix's own triggers fire internally and instantly. BroadcastOS spots state changes over HTTP, so its triggers land roughly 100 to 300 ms later. That's fine for audio fades, overlays and follow on takes. Use vMix's own triggers for anything frame accurate.

**vMix categories.** The category tabs in vMix aren't exposed through its API, so BroadcastOS groups inputs by the `CATEGORY // Name` prefix instead. Naming an input `GFX // LOWER-THIRD` files it under GFX automatically.

**Ten layers.** That's vMix's limit, not one this app adds.

**Unsigned installer.** SmartScreen will warn until the project has a code signing certificate.

## Reporting a problem

The **Log** tab records every command sent to vMix and every reply, which usually shows which end went wrong. Include it when you open an issue, along with what you were doing and which version you're on. The version is shown under File → Check for updates.
