English | [中文](README.md)

<div align="center">

# PinchBridge

**Magic Mouse two-finger pinch → native macOS continuous zoom + global ⌘Q mis-quit guard**

[Download v3.0.0](https://github.com/Vynje/PinchBridge/releases/latest) · [Privacy Statement](PRIVACY.md) · [License](LICENSE.md)

**Free · No ads · No telemetry · Fully local**

![License: Freeware](https://img.shields.io/badge/License-Freeware-E8A03D?style=flat-square) ![Platform](https://img.shields.io/badge/Platform-Apple_Silicon-9AA3B2?style=flat-square) ![Requires](https://img.shields.io/badge/Requires-macOS_14%2B-9AA3B2?style=flat-square) ![Telemetry](https://img.shields.io/badge/Telemetry-None-2EA043?style=flat-square)

</div>

> This is the official PinchBridge repository and use is free of charge; for licensing and redistribution terms, see [LICENSE.md](LICENSE.md).

---

PinchBridge turns the **Magic Mouse two-finger pinch** into **native macOS zoom gesture events**, and while it's at it provides **global ⌘Q mis-press protection** in a small menu bar utility.

Magic Mouse (Gen 1/2) hardware doesn't produce a pinch gesture, so the system layer never receives magnification events and apps like Preview, Safari, and Photos have no pinch-to-zoom — PinchBridge reads two-finger contacts at the touch-surface layer, computes how the spacing changes, and synthesizes system-level zoom gestures so those apps zoom as usual. It also stays resident intercepting bare ⌘Q key presses, so a slip of the hand can't quit an app straight out from under unsaved documents.

> **Status transparency**: this project ships arm64 builds only, and it is **not notarized by Apple**. The first launch requires right-click → Open (see [Installation](#installation) below); if that bothers you, please don't use it.

---

## Features

### ① Magic Mouse two-finger pinch → native zoom

- **Read-only touch surface, event-only output**: it only reads touch frames, via the MultitouchSupport framework; on the output side it only posts gesture events — it never intercepts or swallows any input.
- **Hooks only the Magic Mouse by default**: the native pinch on the built-in trackpad / Magic Trackpad is unaffected (it won't get translated twice into double zooming).
- **Pinch and side-swipe don't fight**: two-finger side-swipes (scrolling) are suppressed through centroid-displacement disambiguation, so they won't falsely trigger zoom; a true pinch is two fingers symmetrically converging/diverging around the centroid, which separates the two naturally.
- **Silky, with no staircase**: 1€ Filter smoothing + continuous per-frame emission (the original Casiez 2012 formulation) — none of the stutter where a dead zone builds up and then jumps a whole step at once; each event's delta carries a clamped safety valve, so an anomalous frame can't emit a giant zoom jump.
- **Momentary touch loss doesn't cut the line**: a bridging window backstops momentary contact loss mid-pinch, so one finger on the Mouse dropping out for an instant won't kill the zoom session.
- **Palm rejection (calibration-based)**: a palm-press contact getting merged into the two-finger pair causes "fake pinch, random zoom" — detection is now a two-layer check on **contact position** alone: palm-edge contacts that stay put for a long time (the anchor criterion), plus an exclusion band drawn along the palm edge, contacts inside which don't count toward pinch detection. The band's direction and width don't come from guesswork thresholds: click "**Palm Calibration…**" in the settings panel and they are measured from your two natural gestures (grip + front pinch, each advanced manually once enough samples are collected), taking effect and being saved immediately; the settings panel also has a **live touch preview canvas**, where contacts filtered out by the anchor or the exclusion band are marked with a red ✕ and the band is drawn shaded — you watch with your own eyes exactly what the engine filters out; if the pinch stops working, lower the strength preset or recalibrate. (The old contact-size gate has been retired: real-device data proved MTTouch.size unreliable — some contacts are NaN/garbage values, and the size spike under pinch pressure overlaps the palm edge; position is the only criterion left that holds up.)

### ② Global ⌘Q mis-press guard

- Intercepts the press and release of **bare ⌘Q** (no Shift/Ctrl/Option/Fn modifiers) so a slip of the hand can't quit the app you're editing in.
- All other combinations — ⌘S, ⌘W, ⌘⇧Q (Quit All Windows), etc. — **pass through untouched**; mouse-driven quit actions in the menu bar / Dock don't travel the keyboard event stream and are likewise unaffected.
- Built-in watchdog: if the system disables the event tap on timeout it gets re-enabled automatically; if physical keyboard events stop arriving, it raises a warning instead of failing silently.

---

## System Requirements

| Item | Requirement |
|---|---|
| Machine | Apple Silicon (**arm64 builds only**; Intel Macs not supported) |
| OS | macOS 14.0 or later |
| Mouse | Magic Mouse (Gen 1 / Gen 2) |
| Permissions | Accessibility + Input Monitoring (the first-launch wizard guides you through it; see below) |

Honest disclosures:

- **Not notarized by Apple**: Gatekeeper will block the first launch. See the installation steps below for how to get past it. The app is signed with one and the same **self-signed code-signing certificate** (PinchBridge Dev): subsequent version upgrades **keep the already-granted state** (the Accessibility / Input Monitoring grants don't need to be given again); only the first install goes through the permission wizard once.
- **Identification criteria come from community reverse engineering**: the Magic Mouse Gen 1/2 device family-number criteria (112/113) come from community reverse-engineering work in upstream open-source projects; Magic Mouse 3 and other variants are **unverified**.

---

## Installation

### Step 1: Drag-and-drop install from the DMG

1. Download the latest `.dmg` from the Releases page, open it, and drag **PinchBridge** into the Applications folder;
2. Because it isn't notarized, for the first launch pick either of:
   - In Applications, **right-click PinchBridge → Open**, then click "Open" once more;
   - or run in Terminal:
     `xattr -d com.apple.quarantine /Applications/PinchBridge.app`.

### Step 2: First launch → permission wizard

Sending pinch events needs the "**Accessibility**" permission; ⌘Q interception needs "**Input Monitoring**" to receive physical keyboard events. The app ships with a **permission wizard** on first launch:

1. The wizard has two buttons that **deep-link in one click** to System Settings → Privacy & Security → the "Accessibility" / "Input Monitoring" panes;
2. In the corresponding pane, click **＋** to add (or simply drag in) **PinchBridge** from `Applications`;
3. Return to the wizard — it **checks both permission states live**; once both permissions are green, protection **activates automatically**, with **no manual restart** of the app needed.

While it runs, the status icon showing up in the menu bar means it's working. To **run automatically at login**: menu bar icon → check "**Launch at Login**" (synced live with System Settings → Login Items; unchecked by default).

---

## Usage

### Menu Bar

Click the menu bar icon:

- **Status area** (not clickable): pinch engine state (armed · device name / paused), the cumulative ⌘Q interception count, and the current Secure Input state (while a password field is focused it shows "Active (⌘Q protection paused)"; see Known Limitations item 1);
- **Toggle rows**: pinch zoom on/off (pauses detection; the process and ⌘Q protection stay up), ⌘Q protection on/off, launch at login (synced live with System Settings → Login Items);
- **Permission Guide…** (shown only while the two permissions aren't all granted): brings up the permission wizard window at any time;
- **Settings…** (⌘,): opens the settings panel (see below);
- **Open Log**: reveals `~/Library/Logs/PinchBridge.log` in Finder;
- **Quit**: ends PinchBridge.

### Settings Panel

A four-section, card-style panel (rewritten in Apple style); everything **takes effect live** — changes are swapped into the detection queues immediately through thread-safe config exchange, with no app restart, **no re-granting of permissions, and no reinstall** (permissions are bound to the app bundle itself; the parameters are just configuration). All labels are in plain language and never expose internal parameter names; the seven tuning sliders from the old version have been withdrawn from the panel, and those parameters now go through the advanced tuning-file channel in the "General" section.

**Pinch Zoom**:

- **Enable Pinch Zoom** toggle: pauses gesture recognition only; ⌘Q protection is unaffected; it comes back on automatically after a relaunch;
- **Sensitivity**, a three-way segmented control (Sensitive / Standard / Robust): how much and how fast your fingers must move before zooming starts — the more sensitive, the earlier it kicks in and the easier to mis-trigger; Standard = the factory default. If the current value has been hand-edited via the tuning file to a non-preset value, the panel selects no segment and says so honestly; selecting a segment is what replaces the value (reads are not clamped and not silently snapped);
- **Smoothness**, a three-way segmented control (Silky / Balanced / Responsive): Silky = a steadier picture while your fingers hold still; Responsive = a faster response on fast pinches; Balanced = the factory default. Behind each preset, the three 1€-filter parameters move as a bound group, with values chosen by measured deterministic feel-contract sweeps: static noise suppression, 63% step response, and zero reverse motion on slow pinches all pass, and the three presets are monotonically ordered along the "steadier ↔ more responsive" axis.

**Palm Rejection** (keeps palm pressure from being taken as a pinch):

- **Enable Palm Rejection** toggle — off = no contact gets filtered (the least overhead while you have no palm-pressure misfires);
- **Strength**, a three-way segmented control (Light / Standard / Strong): the stronger it is, the larger the ignored zone on the palm-root side (an exclusion band width of roughly 20% / 30% = factory / 40%). Measured calibration widths mostly land between presets; the panel shows the palm-zone percentage honestly instead of snapping it to a preset;
- **Palm Zone** read-only row: a plain-language description of the calibration result (e.g. "on the wrist side, about 30% of the surface");
- **Palm Calibration… (recommended as your first step)**: a two-stage wizard — stage 1 "hold the mouse still, as usual", stage 2 "make a pinch gesture at the front of the mouse" — once each stage has collected 30 frames, you click "Next" / "Compute" yourself (no time limit); it then automatically computes the **palm-zone direction** (wrist side / fingertip side) and the **exclusion band width**, taking effect and saving immediately (the result panel shows the direction, the band width, and the sampled frame counts of both stages). Pinch detection is paused during calibration, so the calibration gestures won't fire zooms by mistake; if a stage has too few samples (< 30 frames) it reports "Not enough samples, please retry" — just keep your finger flat on the mouse surface instead of hovering;
- **Live touch preview canvas**: the same two-layer rule set as the engine — contacts that fall inside the exclusion-band shading, or that stay still for a long time (tagged "anchor"), are ignored and marked with a red ✕; the two fingers of an in-progress pinch are exempt under the engine's own session-immunity rule. The preview and gesture recognition use the same rule set, and at isolated instants they may differ by one frame;
- **Misfire troubleshooting**: pinch stopped working (real fingers being swallowed; the ignore zone is too large) → lower the strength or "recalibrate"; palm pressure still zooms (the zone is too small or points the wrong way) → raise the strength or "recalibrate"; after switching hands, grips, or mice, the palm zone has moved — just run the calibration again; no reinstall or re-granting needed;
- The old four-level presets and the size gate have been removed: MTTouch.size has been ruled out (some contacts are NaN/garbage values, and the size spike under pinch pressure overlaps the palm edge), leaving contact position y as the only criterion. Leftover palmSizeGate / palmRatioGate / palmEntryPairCap fields in an old tuning.json are read and ignored, with no effect on operation.

**⌘Q Protection**:

- **Enable ⌘Q Protection** toggle: intercepts keyboard ⌘Q so apps can't be quit by accident; **the toggle state survives restarts** (the same-named toggle in the menu bar shares this state source; flipping either side syncs instantly);
- **Blocked this run** read-only row: the interception count (a press and a release each count as one key event);
- **Secure Input alert** (shown conditionally): while Secure Input is active — password fields and the like — ⌘Q protection is temporarily ineffective; the panel states this openly instead of silently (see Known Limitations item 1);
- **Permission Guide…** (shown conditionally): appears while the "Accessibility + Input Monitoring" pair isn't fully granted; click it to bring up the permission wizard.

**General**:

- **Launch at Login** toggle: synced live with "System Settings → General → Login Items";
- **Open Log**: reveals `~/Library/Logs/PinchBridge.log` in Finder;
- **Per-frame Trace** toggle (**off by default**): when on, writes touch spacing, contact sizes, contact positions on the mouse surface, and filter state to a local CSV frame by frame (for local troubleshooting only, never uploaded; see [PRIVACY.md](PRIVACY.md) for the privacy notes); it takes effect immediately while idle and is applied once a pinch session ends;
- **Advanced Parameters / Show File…**: parameters outside the panel (the bridging window, the per-event clamp cap, the two palm-anchor parameters, etc.) can only be adjusted via `~/Library/Application Support/PinchBridge/tuning.json`; after editing, **restart the app for it to take effect**. Panel commits use **merge writes**: only the keys owned by the panel's controls are rewritten, and other values you hand-edited (custom keys included) are not overwritten;
- **Restore Defaults…**: sensitivity, smoothness, and palm rejection return to factory (exclusion band width 30%, direction rear, calibration results discarded) — the only destructive action in the panel; a confirmation dialog pops up; hand-edited out-of-panel values in the tuning file are unaffected;
- The bottom shows the app version plus a "**Project Home & License**" link row: clicking it opens `github.com/Vynje/PinchBridge` in the system default browser (the app itself initiates no network connections).

---

## Known Limitations (Honest List)

1. **⌘Q protection pauses during Secure Input**: when secure fields such as password boxes are focused (the system activates Secure Event Input), macOS stops delivering keyboard events to third-party event taps, so ⌘Q protection **necessarily goes deaf** — pressing ⌘Q during this period really does quit the app. This is an Apple OS-level restriction; in user space, without root or a DriverKit entitlement there is no plugging it, and this project makes no promise of any remedy. The app detects it by polling at 2 Hz and shows "protection paused / protection restored" in the menu bar status. The pinch feature is **unaffected** (read-only touch surface, gesture events only — it never touches the keyboard stream).
2. **Pinch and two-finger side-swipe coexist**: pan-style side-swipes are suppressed by the centroid gate; but **symmetric converge/diverge side-swipes** (side-swiping while the two-finger spacing drifts past the entry threshold of about ±1.8% with the centroid essentially still) still enter a zoom session, and once the session has started it never returns to the gate — it continues until the fingers lift.
3. **Private API risk**: both the MultitouchSupport framework interfaces and the gesture CGEvent fields are unpublished SPI, and a major macOS update may break them (failure mode: silence, no events).
4. **Magic Mouse 3 unverified**: device identification relies on community-reverse-engineered family numbers (112 = Gen 1, 113 = Gen 2); other/newer models are unverified and may fail to attach by default.
5. **Large pinches at the edge of the touch surface**: once the fingers are pushed to the edge and the pinch continues, the centroid is forced to move and the side-swipe gate may suppress it — lift and re-touch to recover.
6. **Permissions generally survive updates/reinstalls**: the app is signed with the same self-signed certificate, so grants are retained with the signing identity across upgrades; if some update genuinely invalidates them (e.g. the signing identity changed), just delete the old entries in the two permission panes and re-add once (the wizard will guide you again).

---

## Uninstall (Three Steps)

1. Menu bar icon → **Quit** PinchBridge (if "Launch at Login" was ever enabled, turn it off in the menu bar first, or remove PinchBridge under System Settings → Login Items & Extensions);
2. Drag **PinchBridge** from Applications to the Trash;
3. In System Settings → Privacy & Security, in both the **Accessibility** and **Input Monitoring** lists, **manually remove** the PinchBridge entries (uninstalling does not clear the permission records automatically).

Optional cleanup (the actual locations of the config and logs; once you're sure you won't use them again, the whole folders can be deleted):

- Config: `~/Library/Application Support/PinchBridge/tuning.json` (earlier versions used `~/Library/Application Support/pinch-bridge/`);
- Logs: `~/Library/Logs/PinchBridge.log`; per-frame CSVs generated while diagnostic tracing is on also live under `~/Library/Logs/`.

---

## Acknowledgments

This project stands on the shoulders of two MIT open-source projects:

- [ishora](https://github.com/YRDEV1/ishora) (MIT) — the Technique source for the MultitouchSupport private-framework touch-reading implementation, the Magic Mouse family-number criteria, and the centroid swipe-disambiguation idea;
- [MagicMouseZoomBridge](https://github.com/EnesSU/MagicMouseZoomBridge) (MIT) — the Technique source for the gesture CGEvent private field definitions and the event-construction style (including the key finding that adjacent event timestamps must strictly increase).

The full copyright and MIT license texts are in [ACKNOWLEDGMENTS.md](ACKNOWLEDGMENTS.md)
(the license-compliance statement for distributing closed-source binaries).

## Privacy Statement

**Zero network, zero telemetry, fully local**: no network requests are made and no data is collected. For a point-by-point account of what each permission is used for, see [PRIVACY.md](PRIVACY.md).

---

## TL;DR

**PinchBridge** is a small menu bar app for Mac that does two things:

1. **Magic Mouse pinch → native macOS zoom.** The Magic Mouse hardware
   can't produce a pinch gesture, so apps like Preview / Safari never see
   one. PinchBridge reads two-finger contacts at the multitouch layer and
   synthesizes native magnification gesture events, with 1€-filter
   smoothing (no staircase jumps), swipe disambiguation, a bridging
   window against momentary touch loss, and a **palm-rejection layer**
   (a 2-gesture calibration wizard that measures your palm zone, three
   strength presets, and a live touch preview that shows exactly what the
   engine filters) so a resting palm doesn't fake a pinch.
2. **Global ⌘Q guard.** It swallows plain ⌘Q key presses system-wide so
   you don't accidentally quit an app mid-edit. Every other shortcut
   (⌘S, ⌘W, ⌘⇧Q, …) passes through untouched.

**Requirements:** Apple Silicon (arm64-only build), macOS 14+, Magic
Mouse (Gen 1/2). **Not Apple-notarized** — on first launch, right-click
the app → Open, or run `xattr -d com.apple.quarantine
/Applications/PinchBridge.app`.

**Setup:** drag the DMG into Applications, launch once, and the built-in
wizard deep-links you to the Accessibility and Input Monitoring panes;
once both permissions turn green, protection activates automatically —
no restart needed.

**Tuning:** the settings panel edits everything live (no restart, never
invalidates your permissions): semantic presets for pinch sensitivity,
smoothness, and palm-rejection strength, a calibratable palm-rejection
zone (2-gesture wizard + live dual-layer preview that shows exactly what
the engine filters), pinch / ⌘Q guard / launch-at-login toggles in the
menu bar, an optional off-by-default frame trace (local CSV only), and
advanced knobs in a JSON file that panel edits merge into instead of
overwriting.

**Honest limits:** during password fields (Secure Input) the ⌘Q guard is
paused — an Apple OS-level restriction; symmetric spread/squeeze while
side-swiping can still trigger zoom; uses private APIs that a future
macOS may break; Magic Mouse 3 unverified.

**Uninstall:** quit the app, delete it from Applications, and remove its
entries from the Accessibility and Input Monitoring lists in System
Settings.

**Privacy:** no network access, no telemetry, everything stays on your
machine. See [PRIVACY.md](PRIVACY.md).
