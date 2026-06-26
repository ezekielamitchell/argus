# Research Direction

> Status labels used throughout this document:
> **[current]** = demonstrated or in place today ·
> **[planned]** = scheduled engineering, not yet built ·
> **[hypothesis]** = a research conjecture we intend to test, not an assertion of fact.
>
> ARGUS is an early-stage, open-source research program. Today its only concrete
> asset is the CASEset dataset (Phase 1). Everything described below as a model,
> pipeline, or capability is **[planned]** or **[hypothesis]** unless explicitly
> marked **[current]**. This document is about direction and open questions, not
> shipped results.

## Lead Question: What Makes ARGUS Different From Traditional Gaze Estimation?

Traditional gaze estimation answers a narrow, well-posed question: *given an image
of a face or eye, where on the screen (or in the scene) is the person looking?*
The output is a coordinate — a `(x, y)` point, sometimes a 3D gaze vector — and the
field has matured around that target. There are public datasets, established error
metrics (typically angular error in degrees or on-screen error in pixels/cm), and
strong baselines.

ARGUS does not claim to beat those systems at producing a gaze point. Accurate gaze
estimation is a *means*, not the *end*. The question ARGUS investigates is one layer
up:

> *Given where an operator looks, the screen context they are looking at, and the
> temporal structure of how their attention moves, what — if anything — can we
> reliably and ethically infer about their understanding, priorities, and intent,
> and could that inference help an autonomous system support them?*

That reframing changes almost everything about the problem:

| Dimension | Traditional gaze estimation | ARGUS research direction |
| --- | --- | --- |
| Output | A gaze point / vector | An interpretation of attention in context **[hypothesis]** |
| Unit of analysis | A single frame | A temporal sequence of attention over a task |
| Context used | The eye/face image | Eye/face **plus** screen content, interaction logs **[planned]** |
| Success metric | Angular / on-screen error | Whether inferred attention is *useful and trustworthy* to a downstream system **[hypothesis]** |
| Failure mode that matters most | Inaccurate point | Confident-but-wrong interpretation that misleads an operator **[hypothesis]** |

The core hypothesis driving the program is stated as a hypothesis, deliberately:

> **[hypothesis]** Human attention carries information about intent, threat
> awareness, priorities, and situational understanding. If that information can be
> reliably and ethically extracted from low-cost sensing (webcam + screen context
> + interaction logs), it could improve how autonomous systems support human
> operators.

The rest of this document walks the ladder from *gaze* to *attention* to *intent*
to *teaming*, *supervision*, and *cueing*, marking at each rung what is open and
what could go wrong. For where this sits in the program plan, see the
[Roadmap](./ROADMAP.md); for the empirical foundation, see [DATASET.md](./DATASET.md);
for the conceptual data flow, see [ARCHITECTURE.md](./ARCHITECTURE.md).

## From Gaze to Attention

A gaze point is an instantaneous estimate. *Attention* is the interpretation of
gaze in context and over time. The distinction is the conceptual heart of ARGUS.

Consider three operators whose webcams produce an identical gaze point — the same
`(x, y)` on screen. Without context, the three are indistinguishable. With context,
they may be doing very different things:

- one is **fixating** on a flagged object because it is anomalous;
- one is **passing through** that location en route to a control on the far side;
- one is **disengaged**, eyes resting on a region while attention is elsewhere.

Gaze estimation cannot separate these cases by construction — it only reports the
point. Distinguishing them requires combining the gaze stream with *what was on the
screen there* and *the temporal pattern of attention around that moment*.

ARGUS therefore studies attention as a function of multiple synchronized streams,
not a single image:

```
                 +-------------------+
 webcam frames ->|                   |
 screenshots ----|  context-aware    |--> attention interpretation
 gaze coords ----|  attention model  |    (fixations, dwell, revisits,
 interaction ----|   [planned]       |     scan paths, salience-in-context)
   logs          +-------------------+    [hypothesis: usefulness]
```

Concretely, the kinds of derived signals this direction is interested in include
fixations and dwell time, saccade/scan-path structure, revisit patterns to the same
region, and the relationship between where attention lands and *what was rendered
there* (text, an alert, a tracked object, an empty area). These are well-studied
constructs in eye-tracking and human-factors research; ARGUS's contribution is not
to reinvent them but to study them **in operational screen context** using
low-cost sensing, and to ask whether they support the downstream inferences below.

What is **[current]** today: none of these models exist in the repository yet. What
exists is CASEset — synchronized webcam frames, screenshots, gaze coordinates, and
interaction logs — which is the substrate these models would be trained and
evaluated on. Baseline gaze models are **[planned] (Phase 2 — Baseline Gaze
Models)**; context-aware attention models are **[planned] (Phase 3 — Context-Aware
Attention Models)**.

## Operator Intent

If "attention in context" can be estimated, the next rung is whether its *temporal
structure* says anything about operator **intent** — what the operator is trying to
do, what they have noticed, and what they are about to act on. This is **Phase 4 —
Intent Inference [research]**, and every claim in this section is **[hypothesis]**.

Candidate hypotheses worth testing, none yet supported by ARGUS results:

- **[hypothesis]** Repeated revisits to a region before an action correlate with the
  operator treating that region as decision-relevant.
- **[hypothesis]** A sudden change in scan-path entropy (broad scanning collapsing
  to tight fixation) marks a transition from search to target engagement.
- **[hypothesis]** Dwell time on an alert, combined with whether the operator then
  interacts with it, distinguishes "noticed and triaged" from "noticed and
  dismissed" from "never noticed."
- **[hypothesis]** Sequences of attention across UI regions encode a coarse picture
  of the operator's working task model that a system could use to anticipate needs.

The honest framing is that intent inference from attention is *plausible but
unproven here*, and is one of the easiest places to fool yourself. Attention is
correlated with intent, but it is not intent; an operator can look without caring and
care without looking. Section [Key Risks & Unknowns](#key-risks--unknowns) treats
the specific failure modes (spurious correlation, individual variation) that make
this rung hazardous. The methodological commitment is to treat any intent signal as
a *probabilistic, revocable hypothesis about the operator*, never a verdict, and to
evaluate it against held-out operators and tasks rather than in-distribution accuracy
alone (see [EVALUATION.md](./EVALUATION.md)).

## Human-Machine Teaming

Suppose attention *can* be interpreted usefully. The teaming question is what an
autonomous system should do with it. ARGUS's framing treats attention as a **shared-
context channel** — a low-bandwidth, mostly implicit way for an operator and a
machine to stay aligned on *what matters right now* — rather than as a command
interface. This is **Phase 6 — Human-Machine Teaming [research]**.

Why a *channel* and not a *controller*: gaze and attention are noisy, involuntary in
part, and easy to over-interpret. Using them to issue commands ("look here to fire")
is both unsafe and ethically off-limits for this program. Using them as *context*
that a system can weigh — alongside explicit operator actions, which remain
authoritative — keeps the human's deliberate inputs in charge while letting the
machine maintain a model of operator focus.

Possibilities this direction wants to explore, all **[hypothesis]**:

- **[hypothesis]** A system that knows what the operator has *not* yet attended to
  can prioritize bringing those items to notice, reducing missed events.
- **[hypothesis]** Shared attention context lets a system calibrate *when* to
  interrupt — deferring low-priority prompts while the operator is visibly engaged in
  a focused subtask.
- **[hypothesis]** Divergence between where the system "expects" attention and where
  it actually is can flag mismatched mental models worth reconciling.

```
   Operator  --attention (implicit context)-->  ARGUS  ---> Autonomous System
      ^                                                          |
      |<----------- explicit actions stay authoritative ---------|
                   (machine never acts on gaze as a command)
```

The open research problem is not just *can we build this channel* but *should a
given inference flow across it at all* — what is accurate enough, stable enough, and
explainable enough to be allowed to influence machine behavior. That bar is high and
intentionally so.

## Mission Supervision

A second, possibly more tractable, use of attention is as a **supervision and
oversight signal** with the human firmly in the loop — and equally, as a way to keep
the *human's own* oversight honest. This sits across **Phase 6** and **Phase 7 —
Mission Support Systems [research / long-term]**.

The framing inverts the usual "AI watches the human" anxiety. Here, attention data
is mostly used to support the operator's supervisory role over autonomy:

- **[hypothesis]** If autonomy surfaces a recommendation in a region the operator
  demonstrably never attended to, the system can require explicit acknowledgement
  rather than assuming silent assent — guarding against automation-driven
  complacency.
- **[hypothesis]** Aggregate, privacy-preserving attention statistics could help
  evaluate *interface* design (what gets missed, what draws disproportionate focus)
  rather than evaluate individuals.
- **[hypothesis]** Sustained attentional signatures associated with high workload or
  disengagement could prompt the system to slow down, simplify, or hand back
  control.

Two hard constraints bound this entire section. First, **the human stays in the
loop**: attention-derived signals inform and prompt, they do not authorize action,
and they never become a productivity or fitness score applied to a person. Second,
**consent and scope**: any supervisory use of attention requires the consent,
transparency, and oversight commitments described in [ETHICS.md](./ETHICS.md).
Surveillance of operators is an explicit non-goal; the design intent is to make
*autonomy* more accountable to the human, not the reverse.

## Autonomous Systems & Sensor Cueing

The furthest-out direction asks whether **attention priors could bias machine
sensing** — for example, whether the regions and object types a skilled operator
attends to could inform where an autonomous sensor looks, what it prioritizes, or how
it allocates limited bandwidth. This is **Phase 5 — Sensor Cueing [research]** and is
the most speculative rung on the ladder. Treat everything here as open research.

The intuition: a trained operator's attention encodes hard-won priors about what is
worth examining in a given scene or task. If those priors could be distilled into a
usable signal, an autonomous system *might* use them to focus sensing — a soft,
human-informed prior over "where to look next" — rather than scanning uniformly.

Why this is genuinely open, and why we will not overstate it:

- **[hypothesis]** Operator-derived salience generalizes across operators and
  scenes well enough to be useful as a prior. This may simply be false; expert
  attention can be idiosyncratic or task-specific.
- **[hypothesis]** A cueing prior improves a sensing objective (coverage, time-to-
  detect) without introducing dangerous blind spots where humans *systematically*
  fail to look.
- **[open problem]** How to combine a human-attention prior with the system's own
  detectors without the human prior overriding evidence the machine has but the
  human lacked.

Any work here is bounded by the same rules as the rest of the program: **no
exaggerated capability claims, no implication of deployment, no claim of evaluation
by or relationship with any external organization.** Defense-autonomy companies such
as Anduril, Shield AI, Saronic, and Scout AI are referenced elsewhere in the repo
only as *context* for why operator-attention research matters — not as users,
partners, or endorsers of ARGUS. This section describes a research question, not a
product.

## Open Research Questions

These are the questions ARGUS exists to investigate. None are answered yet.

1. **Context lift.** Does adding screen context and interaction logs measurably
   improve attention interpretation over gaze-only baselines, and on what tasks?
   (Phases 2-3)
2. **Temporal structure → intent.** Which temporal attention features, if any,
   carry reliable information about operator intent across *different* operators and
   tasks? (Phase 4)
3. **Generalization.** How much does attention modeling degrade under distribution
   shift — new operators, new interfaces, new lighting/hardware? Is there a stable
   core or is everything operator-specific?
4. **Usefulness vs. accuracy.** What is the right way to measure whether an attention
   signal *helps* a downstream system, as opposed to merely being accurate in
   isolation? (See [EVALUATION.md](./EVALUATION.md).)
5. **Calibrated uncertainty.** Can attention-derived inferences report honest
   confidence, so a downstream system can ignore them when they are unreliable?
6. **Teaming bandwidth.** What is the minimum, most explainable inference that is
   safe to let influence machine behavior through the shared-context channel?
7. **Cueing transfer.** Do operator-attention priors transfer into a useful sensing
   prior, or are they too idiosyncratic to generalize? (Phase 5)
8. **Ethical envelope.** Where exactly is the line between attention-as-support and
   attention-as-surveillance, and how is it enforced technically, not just by policy?
   (See [ETHICS.md](./ETHICS.md).)

## Key Risks & Unknowns

We list these prominently because candor about limitations is a design value of this
program, not a disclaimer to be buried.

- **Individual variation.** Attention patterns differ substantially between people
  and within the same person over time. Models that fit one operator may not
  transfer, and aggregate models may erase the very signal that matters. The size and
  diversity of CASEset is a hard limiter here, and exact dataset statistics are
  treated as `<TBD>` rather than invented (see [DATASET.md](./DATASET.md)).
- **Distribution shift.** Real conditions — hardware, lighting, interface layout,
  task type, fatigue — will differ from any collection setting. Performance under
  shift, not in-distribution accuracy, is the relevant bar, and it is unknown.
- **Spurious correlation.** Attention is correlated with many confounds (UI layout,
  alert placement, habit). An "intent" model may be learning interface artifacts
  rather than anything about the operator. Disentangling these requires deliberate,
  adversarial evaluation, and may not fully succeed.
- **Confidently-wrong inference.** The most dangerous failure is not low accuracy but
  a *confident, wrong* interpretation that an operator or system trusts. Calibration
  and graceful degradation are prerequisites, not enhancements.
- **Automation bias and over-trust.** Surfacing attention-derived prompts could make
  operators defer to the system in exactly the cases where independent human judgment
  matters most. The supervisory framing must be tested for this, not assumed safe.
- **Privacy and consent limits.** Webcam imagery and gaze are sensitive. Some
  otherwise-useful research directions are out of scope on ethical grounds regardless
  of feasibility. The constraints in [ETHICS.md](./ETHICS.md) are binding, not
  aspirational.
- **Sensing-bias blind spots.** If human attention priors bias machine sensing
  (Phase 5), they can import human blind spots — systematic places people fail to
  look — into the autonomous system. This is a safety risk, not just a performance
  one.
- **Early-stage uncertainty.** Most of this program is unbuilt. Any of these
  directions may prove unworkable, and reporting a negative result is an acceptable
  and valuable outcome.

---

*Related documents:* [Project Overview](./PROJECT_OVERVIEW.md) ·
[Roadmap](./ROADMAP.md) · [Architecture](./ARCHITECTURE.md) ·
[Dataset](./DATASET.md) · [Evaluation](./EVALUATION.md) · [Ethics](./ETHICS.md)
