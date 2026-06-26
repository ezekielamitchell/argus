# Ethics

This document states the ethical commitments that govern ARGUS — Adaptive
Reconnaissance and Gaze Understanding System. It is written for engineers,
researchers, reviewers, and collaborators who need to understand not just what
ARGUS studies, but the limits the project places on itself.

ARGUS is an early-stage, open-source research program investigating whether
**operator attention** — where a human operator looks, and the temporal
structure of that attention — can serve as a usable signal for autonomous
systems that support human operators. That research question sits at the
intersection of two areas where ethics cannot be an afterthought: sensing of
human attention, and defense autonomy. This document is the project's standing
answer to "what are you willing, and not willing, to do?"

It is a **living document** (see [Limitations of This Statement](#limitations-of-this-statement)).
It is meant to be read alongside the dataset documentation in
[`DATASET.md`](./DATASET.md) and the research framing in
[`RESEARCH_DIRECTION.md`](./RESEARCH_DIRECTION.md).

---

## Why Ethics Is Central Here

Two facts about ARGUS raise the stakes above those of a typical machine-learning
repository.

**1. The signal is intimate.** Attention is not a neutral measurement. Where a
person looks reveals what they are thinking about, what they are worried about,
what they are ignoring, and sometimes what they intend to do next. The raw
inputs the program studies — webcam imagery, screen context (screenshots), gaze
coordinates, and interaction logs — are among the most sensitive data a person
can produce at a workstation. They can expose a face, a physical environment, on
screen content, reading patterns, and cognitive state. Mishandled, this data is
a vector for surveillance, profiling, and harm.

**2. The intended domain is defense.** ARGUS frames its work in the context of
human-machine teaming and autonomous reconnaissance because that is where
operator-attention signals could matter most. That context carries real
consequences: systems that inform decisions in defense settings can contribute,
directly or indirectly, to the use of force. A research project that ignores
this is being dishonest about its own subject matter.

Neither fact is a reason to abandon the work. Both are reasons to be explicit,
conservative, and accountable about how it is done. The remainder of this
document sets out how.

A note on what ARGUS actually is today: it is **Phase 1 — CASEset Dataset
Foundation [current]**. The end-to-end system implied by the project vision does
not exist. Most components are planned engineering or open research questions.
Throughout this document, claims about future behavior are labeled `[planned]`
or `[hypothesis]`; claims about present behavior are labeled `[current]`. We
hold ourselves to those labels precisely because overclaiming in this domain is
itself an ethical failure.

---

## Privacy

Webcam and screen capture are, together, close to a worst case for privacy: a
camera pointed at a person and a recording of everything on their display.
ARGUS treats that data accordingly.

**Data minimization.** The program collects the minimum needed to answer a
specific research question, not the maximum that the hardware could capture.
Streams that are not required for an experiment are not recorded. Where a derived
feature (for example, a gaze coordinate) is sufficient, we prefer it over the raw
imagery it was derived from.

**On-device and at-source processing where possible.** `[planned]` The
preferred architecture extracts features close to the sensor and avoids moving
raw webcam frames or full screenshots off the capture machine when a lower
fidelity representation will do. The goal is that the most sensitive raw streams
have the shortest possible lifetime and the smallest possible blast radius.

**De-identification.** `[planned]` Where raw imagery must be retained, the
project's direction is to de-identify it as early as is technically feasible —
for example, by reducing screenshots to the regions and features relevant to a
task, and by separating identity from behavioral data. De-identification is
documented as best-effort, not as a guarantee: face and environment imagery are
inherently re-identifiable, and we do not claim otherwise.

**Storage and retention.** Raw human-subject data is stored only as long as a
documented research purpose requires, under access controls, and is git-ignored
and never committed to the repository (see the `data/` directory in the
[repository structure](../README.md#repository-structure) and the handling rules
in [`DATASET.md`](./DATASET.md)). Retention windows, access scope, and deletion
procedures are defined per dataset and recorded in the dataset documentation,
with `<TBD>` placeholders where a specific value has not yet been set rather than
invented numbers.

**Sensitive content on screen.** Because screen capture can incidentally record
unrelated personal information (messages, credentials, third-party data), capture
sessions are scoped to controlled tasks and environments, and participants are
told what will be on screen and recorded before a session begins.

The concrete schema, collection method, and privacy limits of the current
dataset are documented in [`DATASET.md`](./DATASET.md). This section states the
principles; that document states the specifics.

---

## Consent

All human-subject data in ARGUS is collected under **informed, specific, and
revocable** consent. There is no "found" attention data, no scraped webcam
imagery, and no covert collection.

- **Informed.** Participants are told, in plain language, what is captured
  (webcam frames, screenshots, gaze coordinates, interaction logs), why it is
  captured, how it will be stored and for how long, who can access it, and what
  the foreseeable uses are — including that ARGUS is defense-oriented research.
- **Specific.** Consent is tied to a described purpose and scope. Use of data
  for a materially different purpose requires renewed consent, not a broad
  blanket authorization obtained once.
- **Revocable.** Participants may withdraw consent and request deletion of their
  data. The process for withdrawal and the practical limits on deletion (for
  example, data already incorporated into a trained model artifact) are disclosed
  up front, honestly, rather than promised away.

**Participant rights** the project commits to honoring: to understand what is
collected; to decline any individual stream; to withdraw; to request deletion;
and to be told of known risks, including the inherent re-identifiability of face
and environment imagery. Procedures, consent-form templates, and any review or
oversight applicable to a given collection are documented with the dataset in
[`DATASET.md`](./DATASET.md). Where formal ethics review applies to a specific
study, that is recorded there rather than assumed here.

---

## Transparency

ARGUS is open source in part because attention-sensing research should be
inspectable.

- **Open documentation of methods.** Collection procedures, data schemas,
  synchronization methods, model designs, and evaluation protocols are documented
  in the repository so that others can scrutinize and reproduce them. See
  [`DATASET.md`](./DATASET.md) and the evaluation documentation.
- **Honest about limits and failure modes.** We document what a model does *not*
  do, the conditions under which it degrades, and known failure modes — including
  the ways gaze and attention estimates can be wrong (lighting, occlusion,
  calibration drift, individual variation, distribution shift). Known limitations
  are treated as first-class documentation, not fine print.
- **No overclaiming.** ARGUS does not describe aspirations as capabilities. The
  project uses explicit `[current]`, `[planned]`, and `[hypothesis]` labels so a
  reader can never mistake a research direction for a demonstrated result. We do
  not claim any relationship with, endorsement by, or deployment in any
  organization. Reference points to companies working in defense autonomy are
  used only to explain *why the problem space matters*, never to imply
  affiliation.

Reproducibility is valued over impressive-looking numbers. Where a precise
statistic is not established, the documentation says so with a `<TBD>` placeholder
instead of a fabricated figure.

---

## Human Oversight and Meaningful Human Control

ARGUS is conceived as **decision-support for a human operator, never as an
autonomous authority**.

The intended role of the system — even in its fullest `[planned]` form — is to
help a human understand a situation and to surface context, not to act on the
human's behalf in any consequential way. Specifically:

- The human operator stays **in the loop and in command**. ARGUS may model,
  estimate, or cue; it does not decide.
- ARGUS is **never** positioned as a use-of-force authority. It does not select,
  prioritize, or engage targets, and the architecture is not designed to. This
  is an explicit non-goal, restated in
  [Military and Defense Applications](#military-and-defense-applications).
- Outputs are advisory and are intended to be **contestable and overridable** by
  the operator. A design that makes its inferences hard to question or hard to
  ignore would defeat the purpose and is treated as a defect.
- Because attention estimates are uncertain, the project's direction is to
  communicate confidence and known failure modes alongside any inference, so the
  operator can calibrate trust rather than defer blindly.

"Meaningful human control" here means a human who has the information,
authority, and time to make the decision — not a human whose only role is to
click "confirm" on a recommendation they cannot evaluate. Preserving that
condition is a design constraint on every `[planned]` component.

---

## Military and Defense Applications

ARGUS is defense-oriented research, and this document engages that fact directly
rather than around it.

**Intended use.** The intended application is **supportive and teaming-oriented**:
helping a human operator maintain situational understanding, reducing cognitive
load, and improving how an autonomous system can assist — not replace — that
operator. The conceptual direction is *Operator → Attention → ARGUS → Context
Understanding → Mission Support*, with the human as the decision-maker
throughout. The motivation and research questions are set out in
[`RESEARCH_DIRECTION.md`](./RESEARCH_DIRECTION.md).

**Dual-use reality.** Attention modeling and gaze estimation are dual-use. The
same techniques that help an operator stay aware could, in other hands, be turned
toward surveillance or coercion. We do not pretend this tension away. Open
documentation, the consent and privacy commitments above, and the explicit limits
below are the project's response to it — not a claim that the risk is zero.

**Lines this project will not cross.** These are commitments, not aspirations:

- **No autonomous targeting or engagement.** ARGUS will not be built to select,
  prioritize, or engage targets, and will not serve as an autonomous use-of-force
  authority. Lethal and force decisions remain with accountable humans, outside
  the scope of what this project develops.
- **No covert surveillance of non-consenting individuals.** ARGUS will not be
  used to collect attention, webcam, or screen data from people who have not given
  informed, specific, revocable consent. The techniques are not to be repurposed
  for monitoring people without their knowledge.
- **No deception about capability.** The project will not represent ARGUS as
  operationally validated, deployed, or more capable than it is. Overstating a
  defense system's capability is a safety problem, not just a marketing one.
- **No removal of the human.** Work that aims to take the human out of the
  decision loop is outside the scope of this project as defined here.

If a proposed use of ARGUS conflicts with these lines, the correct outcome is to
decline the use, not to quietly relax the line.

---

## Responsible Deployment

ARGUS has nothing to deploy today; this section states the bar that `[planned]`
components must clear before they could responsibly support a human operator.

- **Red-teaming.** Models and pipelines should be adversarially probed for
  failure: spoofed or manipulated gaze, distribution shift, degraded sensors, and
  attempts to make the system assert confidence it has not earned. Findings are
  documented, not buried.
- **Bias evaluation.** Gaze and attention estimation can perform unevenly across
  faces, eyewear, lighting, ergonomic setups, and individual physiology.
  Evaluation should measure performance across relevant subgroups and conditions,
  and report disparities. The evaluation protocol lives in
  [`EVALUATION.md`](./EVALUATION.md).
- **Failure handling.** The intended behavior under uncertainty is to **degrade
  safely** — to abstain, to widen confidence, or to defer to the operator —
  rather than to emit a confident wrong answer. Silent failure is treated as the
  most dangerous failure.
- **Accountability.** Decisions remain attributable to people. The project
  maintains documentation, versioning, and provenance so that a given output can
  be traced to the data, model, and configuration that produced it. The
  responsible point of contact for this repository is its maintainer,
  Ezekiel A. Mitchell.

These are entry conditions for responsible use of any future capability, not a
checklist that has been completed.

---

## Limitations of This Statement

This is a **living document**, and it has real limits:

- It states commitments and intent; it is **not, by itself, a guarantee** that
  every commitment is perfectly implemented in code today. Where the gap between
  principle and implementation matters, the project's direction is to document
  the gap rather than hide it.
- ARGUS is early-stage. Many of the safeguards described here apply to components
  that are `[planned]` and not yet built. The honest statement is that the
  *commitments* exist now and constrain what gets built; the *mechanisms* are
  partly future work.
- This document does not substitute for applicable law, institutional review, or
  the specific consent and oversight arrangements of any individual study. Those
  are recorded with the relevant dataset in [`DATASET.md`](./DATASET.md).
- It will need revision as the technology, the threat model, and the surrounding
  norms evolve. Outdated ethics documentation is itself a risk, so this file is
  expected to change.

Questions, concerns, or proposed revisions should be raised through the
repository's issue tracker or to the maintainer. Surfacing a problem with this
statement is a contribution, not a nuisance.

---

*Cross-references:* [`DATASET.md`](./DATASET.md) ·
[`RESEARCH_DIRECTION.md`](./RESEARCH_DIRECTION.md)

*License: MIT, © 2026 Ezekiel A. Mitchell.*
