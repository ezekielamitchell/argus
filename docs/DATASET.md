# CASEset — Dataset Documentation

CASEset is the empirical foundation of the ARGUS program (Phase 1 — CASEset
Dataset Foundation **[current]**). This document specifies what CASEset
contains, how it is collected, how its streams are synchronized, how it is
stored on disk, and the privacy and reproducibility constraints that govern its
use.

> **Scope note.** ARGUS is an early-stage, open-source research program. CASEset
> is currently the program's only concrete asset. Everything in later phases
> (baseline models, context-aware attention, intent inference) is planned
> engineering or a research hypothesis and is documented elsewhere. This file
> describes the dataset only.

> **Placeholder convention.** Where an exact figure (participant count, frame
> count, duration, resolution, frame rate, tolerance) has not been fixed and
> recorded, this document uses an explicit `<TBD>` placeholder. Do not infer or
> fill these with estimated values; replace them only with measured numbers from
> an actual collection run.

---

## 1. Overview

**What CASEset is.** CASEset is a multi-stream, time-synchronized recording of a
human operator working at a screen-based task. Each session captures, in
parallel:

1. **Webcam imagery** of the operator's face and head.
2. **Screen context** (screenshots of what the operator was looking at).
3. **Gaze coordinates** (where on the screen the operator was estimated to be
   looking).
4. **Interaction / event logs** (mouse, keyboard, and task-level events with
   timestamps).

The defining property of the dataset is **synchronization**: all four streams
are aligned to a common clock so that, for any moment in a session, you can
recover the operator's face image, the on-screen content, the gaze point, and
the concurrent interaction events together.

**Why it exists.** ARGUS investigates a hypothesis: that operator attention —
where a human looks and the temporal structure of that looking — carries usable
information about intent, threat awareness, priorities, and situational
understanding **[hypothesis]**. Testing that hypothesis requires data in which
attention signals are paired with the context that produced them. CASEset is
built to be that substrate: low-cost sensing (a commodity webcam plus screen and
interaction capture) recorded under documented, reproducible conditions.

**What CASEset is not.** It is not a production telemetry feed, not a deployed
system's output, and not a large-scale benchmark. It is a research dataset,
collected at small scale under lab-like conditions, with the limitations
documented in [Section 9](#9-limitations--biases).

Related documents:

- Privacy, consent, and responsible-use obligations: [ETHICS.md](./ETHICS.md)
- Metrics and the protocol that consumes this dataset: [EVALUATION.md](./EVALUATION.md)

---

## 2. Data Streams

A CASEset session is composed of four streams. Each stream has its own native
sample rate and is timestamped against the session's common clock (see
[Section 4](#4-synchronization)).

### 2.1 Webcam frames

- **Content.** RGB frames of the operator captured by a single commodity webcam
  positioned near the top of the display.
- **Purpose.** Source signal for face/eye detection, head-pose estimation, and
  appearance-based gaze estimation.
- **Native rate.** `<TBD>` fps (target is steady; actual rate is logged
  per-frame rather than assumed).
- **Resolution.** `<TBD>` (record the camera's actual capture resolution; do not
  upsample).
- **On disk.** Stored as individual image files (or an encoded video plus a
  frame-index sidecar — see [Section 6](#6-storage-format--directory-layout)).
  Each frame carries a monotonic frame index and a capture timestamp.
- **Sensitivity.** Contains the operator's face. Treated as personally
  identifying. See [Section 7](#7-privacy-considerations).

### 2.2 Screenshots (screen context)

- **Content.** Periodic captures of the operator's screen — the visual context
  the operator was attending to.
- **Purpose.** Lets gaze coordinates be interpreted against what was actually on
  screen (e.g., which UI element, region, or stimulus a gaze point falls on).
  This is the substrate for context-aware modeling in later phases **[planned]**.
- **Native rate.** `<TBD>` (screenshots may be captured at a lower rate than the
  webcam, or on screen-change events; the chosen policy must be recorded in the
  manifest).
- **Resolution / geometry.** Full display resolution `<TBD>`, with the
  screen-pixel coordinate origin and axis orientation documented so gaze points
  map unambiguously onto screenshots ([Section 5](#5-labels--annotations)).
- **Sensitivity.** May contain arbitrary on-screen content, including text and
  third-party material. Treated as sensitive. See
  [Section 7](#7-privacy-considerations).

### 2.3 Gaze coordinates

- **Content.** Estimated point of regard — where the operator was looking —
  expressed in screen-pixel coordinates, plus (where available) a validity flag
  and a per-sample confidence.
- **Source.** Produced by the gaze sensing used during collection (e.g., an
  eye-tracking device or a software estimator). The exact source and its native
  accuracy must be recorded per session; CASEset treats gaze as a **measured
  input stream**, and its accuracy is a property of the capture setup, not a
  ground-truth guarantee.
- **Native rate.** `<TBD>` Hz.
- **Coordinate frame.** Screen-pixel coordinates by default; an angular
  representation may be derived (see [Section 5](#5-labels--annotations)).
- **Validity.** Samples may be missing or invalid (blinks, off-screen gaze,
  tracking loss). Invalid samples are flagged, not silently dropped or
  interpolated, so downstream code can decide how to handle them.

### 2.4 Interaction / event logs

- **Content.** Time-stamped interaction events: mouse movement and clicks,
  keyboard activity (recorded as event metadata, **not** raw keystroke content —
  see privacy), scrolls, focus changes, and task-level markers (task start/end,
  stimulus onset, prompts).
- **Purpose.** Provides the temporal behavioral structure that complements gaze:
  what the operator did, and when, relative to where they looked.
- **Native rate.** Event-driven (irregular), each event carrying its own
  timestamp.
- **On disk.** A line-delimited JSON log (`events.jsonl`), one event per line,
  ordered by timestamp.

---

## 3. Collection Methodology

> The procedure below is the **generic, reproducible protocol**. Specific
> quantities (counts, durations, hardware models) are recorded per collection
> run in each session's `manifest.json` and in a top-level dataset card; they
> are not hard-coded here. Unknown values appear as `<TBD>`.

### 3.1 Session setup

- **Participant.** One operator per session. Recruitment, eligibility, and the
  consent process are governed by [ETHICS.md](./ETHICS.md); a session is only
  valid if informed consent was recorded before any capture.
- **Environment.** A documented, lab-like setting: fixed seating, a single
  display, controlled lighting `<TBD>`, and a webcam at a documented position
  relative to the screen.
- **Hardware.** Webcam model `<TBD>`, display model and size `<TBD>`, gaze
  sensing device/software `<TBD>`. Recorded per session so setup-dependent
  effects can be analyzed.
- **Software.** The capture tooling lives under `caseset/` in the repository and
  is the authoritative reference for capture behavior; its version (git commit)
  is written into the manifest.

### 3.2 Calibration

- **Gaze calibration.** Before the task, the operator performs a calibration
  routine (e.g., fixating a sequence of on-screen targets) so screen-pixel gaze
  estimates are aligned to the display. The calibration target layout, number of
  points `<TBD>`, and any post-calibration validation error `<TBD>` are recorded.
- **Validation.** Where supported, a separate validation pass measures residual
  gaze error after calibration; the result is stored in the manifest so sessions
  with poor calibration can be filtered.
- **Geometry.** Screen dimensions in pixels and (where known) physical size and
  viewing distance `<TBD>` are recorded to permit later conversion between
  screen-pixel and angular representations.

### 3.3 What a recording session involves

1. Obtain and record informed consent ([ETHICS.md](./ETHICS.md)).
2. Configure and document the hardware/software setup.
3. Run gaze calibration and validation; record residual error.
4. Start synchronized capture of all four streams against the common clock.
5. The operator performs the task(s) for the session `<TBD>` duration.
6. Stop capture; finalize timestamps and write the session manifest.
7. Run the de-identification / review step before the session is eligible for
   sharing ([Section 7](#7-privacy-considerations)).

Each session is self-contained: it can be loaded, inspected, and evaluated
without reference to any other session.

---

## 4. Synchronization

Synchronization is the core technical requirement of CASEset. The streams are
captured by different subsystems at different rates, so each must be placed on a
**common timeline** to be useful together.

### 4.1 Common clock

- All streams are timestamped against a single monotonic clock for the session
  (a monotonic source is preferred over wall-clock time to avoid drift and
  clock-adjustment artifacts).
- The clock source and its units (milliseconds vs. microseconds) are recorded in
  the manifest. Wall-clock session start is stored separately for human
  reference; alignment math uses the monotonic timeline.
- Every sample in every stream stores an absolute timestamp on this timeline,
  **not** an inferred index-based time. Rates are treated as nominal; true
  timing comes from the stored timestamps.

### 4.2 Why alignment matters

The research question is about the relationship between **where someone looks**
and **what is on screen** and **what they do**. A small misalignment between
gaze and screenshot can attribute a gaze point to the wrong UI element; a
misalignment between gaze and interaction events can scramble cause and effect.
Alignment quality therefore bounds the validity of every downstream analysis,
which is why timestamps — not assumed frame rates — are authoritative.

### 4.3 Alignment method and tolerances

- **Pairing.** To associate samples across streams (e.g., the screenshot
  "current" at a given gaze sample), use nearest-timestamp matching with a
  documented maximum allowed time gap. Matches outside the tolerance are marked
  unmatched rather than forced.
- **Tolerance.** Maximum acceptable cross-stream offset: `<TBD>` ms. This value
  must be measured for the capture setup and recorded; it should be small
  relative to the fastest stream's sample interval.
- **Drift.** Any measured clock drift between subsystems and the correction
  applied (if any) are documented per session `<TBD>`.
- **Reporting.** The synchronization quality achieved (e.g., distribution of
  cross-stream offsets) is reported with the dataset so consumers can judge
  fitness for a given analysis. Evaluation protocols that depend on alignment
  are described in [EVALUATION.md](./EVALUATION.md).

---

## 5. Labels & Annotations

CASEset distinguishes between values that are **measured/derived during capture**
and values that are **annotated after the fact**.

### 5.1 Measured vs. derived vs. annotated

- **Measured (capture-time).** Webcam frames, screenshots, raw gaze samples,
  interaction events, timestamps. These are recorded directly.
- **Derived (computed).** Quantities computed deterministically from measured
  streams — e.g., fixation/saccade segmentation from the gaze stream, the
  screenshot region under a gaze point, or angular gaze from screen-pixel gaze.
  Derived values are reproducible from raw data and the documented procedure;
  they are stored as derived artifacts, never confused with raw measurements.
- **Annotated (human-labeled).** Any labels added by a human (e.g., task phase,
  region-of-interest definitions, event semantics). The annotation procedure,
  label schema, and (where applicable) inter-annotator agreement are documented
  alongside any annotations. **[planned]** for annotation layers beyond the
  capture-time markers.

> CASEset does not claim a gold-standard "ground-truth gaze" label. Gaze is a
> measured input whose accuracy is bounded by calibration and the sensing device
> ([Section 2.3](#23-gaze-coordinates)). Treat it accordingly in evaluation.

### 5.2 Gaze coordinate frames

Gaze may be represented in more than one frame; each session documents which
representations are present and how to convert between them.

- **Screen-pixel coordinates.** Origin and axis orientation must be stated
  explicitly. Convention used by CASEset: origin at the **top-left** of the
  primary display, `x` increasing rightward, `y` increasing downward, in display
  pixels. This matches the screenshot pixel grid so gaze points overlay screen
  context directly.
- **Normalized coordinates.** Screen-pixel coordinates divided by display
  width/height, giving values in `[0, 1]`; convenient for resolution-independent
  analysis.
- **Angular coordinates.** Visual angle (degrees) relative to a reference. This
  representation requires display physical size and viewing distance; where those
  are `<TBD>`, angular values cannot be derived and must not be fabricated.

The conversion parameters (display pixel dimensions, physical size, viewing
distance) live in the manifest so any representation can be regenerated.

---

## 6. Storage Format & Directory Layout

CASEset uses a **per-session folder** layout. Each session is a self-describing
directory containing a manifest plus one artifact group per stream. The layout
below is the reproducible on-disk schema.

> **Where this lives.** The tree below describes the dataset **as it sits on
> disk**, rooted under the repository's git-ignored `data/` directory
> ([Section 7](#7-privacy-considerations)). This is **distinct** from the
> tracked `caseset/` directory in the repository, which holds only
> collection/sync/schema **tooling** — never participant recordings. Raw frames,
> screenshots, gaze, events, and manifests for actual sessions exist only under
> `data/` (or external storage) and are never committed.

```
data/                                  # git-ignored artifacts root (see Section 7)
└── caseset/                           # on-disk dataset — NOT the tracked caseset/ tooling dir
    └── sessions/
        ├── session_0001/
        │   ├── manifest.json          # session metadata + provenance (see 6.2)
        │   ├── webcam/
        │   │   ├── frames/            # frame_000001.png, frame_000002.png, ...
        │   │   │                      #   (or webcam.mp4 + frames.csv index)
        │   │   └── frames.csv         # frame_index, timestamp_ms, file, [valid]
        │   ├── screen/
        │   │   ├── shots/             # shot_000001.png, ...
        │   │   └── shots.csv          # shot_index, timestamp_ms, file
        │   ├── gaze.csv               # timestamp_ms, x_px, y_px, valid, confidence
        │   ├── events.jsonl           # one JSON event per line, timestamp-ordered
        │   └── calibration.json       # calibration targets + residual error
        ├── session_0002/
        │   └── ...
        └── DATASET_CARD.md            # dataset-level summary, counts, license
```

Notes:

- **Image vs. video for webcam.** Either individual frame files **or** an encoded
  video with a `frames.csv` index is acceptable; the manifest records which.
  Lossy encoding parameters, if used, are documented (they affect appearance-based
  models).
- **Tabular streams** (`gaze.csv`, `frames.csv`, `shots.csv`) use a header row
  and an explicit `timestamp_ms` column on the common clock.
- **`events.jsonl`** is line-delimited JSON so it can be streamed and appended;
  each line is a complete event object.
- **Determinism.** File naming is zero-padded and monotonic so directory order
  matches temporal order.

### 6.1 Stream file schemas

`gaze.csv` columns:

| column         | type    | description                                             |
| -------------- | ------- | ------------------------------------------------------- |
| `timestamp_ms` | int     | sample time on the session common clock                 |
| `x_px`         | float   | gaze x in screen pixels (top-left origin)               |
| `y_px`         | float   | gaze y in screen pixels (top-left origin)               |
| `valid`        | bool    | `false` for blinks / tracking loss / off-screen         |
| `confidence`   | float   | per-sample confidence in `[0, 1]` if the sensor reports |

`frames.csv` / `shots.csv` columns: `index` (int), `timestamp_ms` (int),
`file` (relative path), and an optional `valid` flag.

`events.jsonl` — each line, for example:

```json
{"timestamp_ms": 102345, "type": "mouse_click", "button": "left", "x_px": 812, "y_px": 437}
{"timestamp_ms": 102500, "type": "task_marker", "name": "stimulus_onset", "task_id": "t3"}
{"timestamp_ms": 103010, "type": "key_event", "action": "keydown", "key_category": "navigation"}
```

> Keyboard events record **category/metadata only** (e.g., `navigation`,
> `text_entry`), never raw typed content. See
> [Section 7](#7-privacy-considerations).

### 6.2 `manifest.json` fields

Every session directory contains a `manifest.json`. Required and recommended
fields:

| field                     | type   | description                                                                 |
| ------------------------- | ------ | --------------------------------------------------------------------------- |
| `schema_version`          | string | version of this on-disk schema                                              |
| `session_id`              | string | unique session identifier (e.g., `session_0001`)                            |
| `participant_id`          | string | **pseudonymous** participant code — never a real name or direct identifier  |
| `consent_recorded`        | bool   | whether informed consent was captured before recording ([ETHICS.md](./ETHICS.md)) |
| `created_utc`             | string | ISO-8601 wall-clock session start (human reference only)                    |
| `clock`                   | object | `{ "source": "...", "units": "ms" }` describing the common monotonic clock  |
| `duration_ms`             | int    | total session duration on the common clock                                  |
| `hardware`                | object | webcam, display, gaze-sensing device/software identifiers `<TBD>`           |
| `capture_software`        | object | `caseset/` tooling name + git commit used to record the session             |
| `webcam`                  | object | storage mode (frames/video), nominal fps `<TBD>`, resolution `<TBD>`        |
| `screen`                  | object | capture policy (rate or on-change), display pixel dimensions `<TBD>`        |
| `gaze`                    | object | source, nominal rate `<TBD>`, coordinate frame, validity semantics          |
| `display_geometry`        | object | pixel dimensions, physical size `<TBD>`, viewing distance `<TBD>`           |
| `calibration`             | object | reference to `calibration.json`, residual error `<TBD>`                     |
| `sync`                    | object | tolerance_ms `<TBD>`, measured offset stats, drift correction notes          |
| `streams`                 | object | relative paths to each stream's files                                       |
| `deidentification`        | object | what de-id steps were applied before the session was made shareable          |
| `license`                 | string | `MIT` (see repository `LICENSE`)                                            |
| `notes`                   | string | free-text anomalies (dropped frames, recalibration, etc.)                   |

The `DATASET_CARD.md` at the dataset root aggregates per-session facts (number
of sessions `<TBD>`, total frames `<TBD>`, total duration `<TBD>`, demographic
summary `<TBD>`) and restates the license and privacy terms. Populate it from
measured session data, not estimates.

---

## 7. Privacy Considerations

CASEset contains **faces** (webcam) and **arbitrary screen content**
(screenshots). Both are sensitive. The obligations below are summarized here and
specified in full in [ETHICS.md](./ETHICS.md); where the two documents overlap,
[ETHICS.md](./ETHICS.md) is authoritative.

- **Consent first.** No stream is recorded without prior informed consent.
  Consent scope (including whether data may be shared beyond the original study)
  is recorded and honored. Participants can withdraw per the process in
  [ETHICS.md](./ETHICS.md), which requires their session(s) to be removable.
- **Pseudonymous identifiers.** Sessions are keyed by a pseudonymous
  `participant_id`. Any mapping from pseudonym to real identity, if it exists at
  all, is kept **out of the repository** and out of the dataset.
- **De-identification.** Before a session is eligible for sharing, it passes a
  documented review/de-identification step (e.g., screening screenshots for
  incidental personal or third-party information; handling of faces per the
  consent scope). The steps applied are recorded in the manifest's
  `deidentification` field.
- **Keystroke minimization.** Interaction logs store key **categories/metadata**,
  never raw typed text, to avoid capturing passwords or message content.
- **Never commit raw data to git.** Webcam frames, screenshots, and any
  participant-level recordings must **never** be committed to the repository. The
  `data/` directory is git-ignored and is the only place local datasets and
  artifacts live. The repository holds **tooling, schema, and documentation** —
  not subject data.

> **Hard rule.** If you are unsure whether a file contains personal data, treat
> it as if it does and keep it out of version control.

---

## 8. Versioning & Provenance

- **Schema version.** The on-disk schema is versioned (`schema_version` in each
  manifest). Changes that alter field meaning increment the version.
- **Tool provenance.** Each session records the `caseset/` capture-tooling git
  commit, so a session can be traced to the exact code that produced it.
- **Immutability.** Released sessions are treated as immutable; corrections are
  made as new versions with documented changes rather than silent edits.

---

## 9. Limitations & Biases

CASEset's limitations are stated plainly because honest scope is more useful than
inflated claims.

- **Small scale.** The dataset is collected at small scale (`<TBD>` sessions,
  `<TBD>` participants). It supports method development and feasibility study,
  **not** broad statistical claims.
- **Demographic bias.** A small participant pool is unlikely to represent the
  range of human appearance, eye physiology, or eyewear. Appearance-based gaze
  estimation is known to be sensitive to such factors, so models trained on
  CASEset may not generalize across demographics.
- **Setup bias.** Fixed lighting, seating, single display, and a specific webcam
  position make the data internally consistent but narrow. Results may not
  transfer to other hardware or ergonomics.
- **Lab vs. field gap.** Sessions are recorded under controlled, lab-like
  conditions. Real operational environments differ in lighting, motion,
  distraction, stress, and display content. CASEset does not capture that gap,
  and ARGUS does **not** claim field validity from lab data **[hypothesis]**.
- **Sensor limits.** Commodity webcams have limited resolution and dynamic
  range; gaze estimates carry calibration-bounded error and dropouts (blinks,
  off-screen, tracking loss). Gaze is a measured input, not ground truth.
- **Task specificity.** Findings are conditioned on the specific task(s) used in
  collection and may not extend to other tasks.
- **Synchronization residual.** Even after alignment, a non-zero cross-stream
  offset remains (`<TBD>` ms); analyses sensitive to fine timing must account
  for it.

These limitations directly shape valid evaluation; see
[EVALUATION.md](./EVALUATION.md) for how metrics are interpreted given them.

---

## 10. Reproducibility Checklist

A session/dataset is reproducible under CASEset if all of the following hold:

- [ ] Informed consent recorded **before** capture; consent scope stored
      ([ETHICS.md](./ETHICS.md)).
- [ ] `manifest.json` present and complete for every session (Section 6.2).
- [ ] Hardware and `caseset/` capture-software version (git commit) recorded.
- [ ] Common clock source and units documented; every sample carries an absolute
      timestamp on that clock.
- [ ] Calibration procedure and residual error recorded (`calibration.json`).
- [ ] Gaze coordinate frame stated (origin, axes, units); conversion parameters
      stored where angular representation is provided.
- [ ] Stream file schemas match Section 6.1 (headers, column types, validity
      flags present).
- [ ] Synchronization tolerance and achieved offset statistics reported; matches
      outside tolerance marked unmatched, not forced.
- [ ] Invalid/missing samples flagged, not silently dropped or interpolated.
- [ ] De-identification step applied and recorded before any sharing.
- [ ] No raw subject data (faces, screenshots, participant recordings) committed
      to git; all such data under the git-ignored `data/` directory.
- [ ] `DATASET_CARD.md` populated from measured values; no fabricated statistics;
      unknowns left as explicit `<TBD>`.

---

## See Also

- [ETHICS.md](./ETHICS.md) — consent, privacy, oversight, and responsible use
  (authoritative on privacy obligations).
- [EVALUATION.md](./EVALUATION.md) — metrics and the protocol that consumes
  CASEset.
- Repository `LICENSE` — MIT, © 2026 Ezekiel A. Mitchell.
