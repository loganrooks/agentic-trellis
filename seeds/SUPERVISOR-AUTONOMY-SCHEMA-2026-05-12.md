# Supervisor autonomy schema — working hypothesis

**Date:** 2026-05-12
**Origin:** Plugin design conversation in supervisor session `e40799cd-85dc-44d9-be4a-70a6c443e20a`, after the failure log forced the question: "where can supervisor autonomy be safely expanded?"
**Status:** Working hypothesis. Will be operationalized across agentic-mail v0.1.x, agentic-ops follow-ups, and f1-modelling onboarding to test fit.
**Companion artifact:** `SUPERVISOR-FAILURES-2026-05-12.md` (the evidence base this schema is provisionally built on).

## Purpose

Provide an analytical framework for *autonomy-expansion decisions* in AI-agential development: when contemplating granting an agent (here, a Claude supervisor over a Codex executor) more authority, longer unattended runs, broader repo scope, or new action types — what determines whether the expansion is safe, and what determines that we don't know?

The framework is **descriptive, not prescriptive**. It does not set autonomy thresholds. It decomposes the decision into evaluable parts and surfaces what we know and don't know about each. Threshold-setting is operator judgment informed by the schema, not output of the schema.

## The four failure-intrinsic dimensions

A failure mode is not a point with properties; it is a **trajectory through stages**, and the dimensions are the probabilities along that trajectory plus the magnitude that survives it. The expected residual harm of a failure mode, given an autonomy configuration, factors as:

```
Expected residual harm = P(occur) × P(undetected | occur) × P(unrecovered | detected) × Impact(uncorrected)
```

The four dimensions are:

1. **Occurrence probability — P(occur)**
   Given the current autonomy and conditions, how likely is the failure to be triggered? Measured per autonomy window (per session, per phase, per action class).

2. **Undetection probability — P(undetected | occur)**
   Given it occurs, how likely is it to escape every detection layer (supervisor self-check, machine guard, post-hoc reconciliation, external observer)? A failure with P(undetected) ≈ 1 is silent — it ships unless prevented at the occurrence stage.

3. **Unrecovery probability — P(unrecovered | detected)**
   Given detection, how likely is recovery to fail or be only partial? Irreversibility is the degenerate case where this equals 1. Most failures have low values here; the ones that don't are categorically different (see haunting #1).

4. **Residual impact — Impact(uncorrected)**
   What damage remains after best-effort recovery? Includes financial, reputational, trust-related, legal, downstream-drift (compounding) effects. Not necessarily monetary; some impact terms are categorical (lost trust does not denominate in dollars).

The dimensions are **conditionally chained**, not independent. Each is conditional on the prior in the sequence. This is the bow-tie analysis pattern from process-safety engineering: causes funnel into the event on the left, consequences fan out on the right, barriers sit at each junction.

## The three companion ontologies

The four dimensions are *failure-intrinsic*. They do not capture everything that bears on the autonomy decision. Three other ontologies complete the picture:

### Mitigation-system ontology

Properties of the guards, hooks, disciplines, and recovery procedures the project has built around failures:

- **Coverage** — which failure modes this mitigation reduces (and at which stage: prevent occurrence, improve detection, accelerate recovery, limit impact)
- **Cost** — development, maintenance, false-positive burden, operational complexity
- **Robustness** — does the mitigation degrade under load, context-window pressure, or session boundaries
- **Auditability** — can we verify the mitigation fired correctly after the fact

Example: the user-side hook that catches URL fabrication (F1 in the failure log) is a mitigation. Its coverage is P(undetected) for that specific failure class; its cost is low; its robustness is high (it runs deterministically); its auditability is full (every block leaves a trace).

### Operating-context ontology

Properties of the environment the project runs in that modulate the failure-intrinsic dimensions:

- **Adversarial pressure** — multiplies P(occur) for any failure exploitable by a motivated bad actor; mostly zero in private experimental work, non-zero the moment external contribution is accepted
- **Regulatory regime** — multiplies Impact for failures that violate compliance, even when underlying technical damage is small
- **Audience composition** — who sees the project, who sees its failures (these can differ); shapes the reputational component of Impact
- **Time sensitivity** — tightens effective P(undetected) by imposing a latency budget on detection
- **Multi-tenancy** — converts per-instance Impact into blast-radius-multiplied Impact
- **Accountability assignment** — who bears responsibility when a failure ships; affects mitigation incentives, not the failure itself

Context modulators are *not* failure properties. The same failure mode has different effective risk in different contexts. The consultant's job at onboarding includes identifying which modulators are active.

### Epistemic ontology

Properties of *what the supervisor knows about itself and the failure registry*:

- **Calibration** — how well-correlated is the supervisor's confidence in its own performance with its actual performance? Low calibration means the supervisor's risk estimates should be discounted, and external corroboration becomes mandatory rather than optional.
- **Registry coverage** — what fraction of the actual failure space is represented in the failure log? This is fundamentally unknowable in absolute terms but can be estimated by how often new failure modes appear during operation.
- **Recency** — when were the estimates last updated against fresh evidence?
- **Provenance** — what is each estimate based on (N=? instances, what project types, what context)?

The epistemic ontology is meta: it tells you *how much to trust the rest of the schema's outputs.* A schema running on low-calibration, low-coverage, stale estimates is a schema producing confident answers from thin air.

## How the autonomy decision composes

The decision is not "score each failure, sum, threshold." It is a multi-ontology composition:

1. **For each known failure mode**, estimate the four failure-intrinsic dimensions under the proposed autonomy configuration.
2. **For each estimate**, mark the relevant mitigations from the response-system ontology and note their coverage and robustness.
3. **For the project's current context**, apply the operating-context modulators to adjust effective values.
4. **Across all of the above**, attach the epistemic ontology — confidence per estimate, coverage of the registry overall.
5. **Surface the dominant terms.** The autonomy decision is usually constrained by one or two dimensions, not by the average. F4 (silent miss) was a P(undetected)-dominated failure; F1 (URL fabrication) is a mitigation-dependent failure; an irreversible failure is a P(unrecovered) = 1 special case.
6. **Operator judgment** sets the autonomy threshold given the surfaced dominant terms.

Note that step 6 is *not the schema's job*. The schema describes risk; the threshold is operator choice.

## On the metaphor

The earlier framing used "columns" — a flat table where each failure is a row, each property is a column. This was wrong in three ways:

- **Conditional structure was lost.** P(undetected) is conditional on P(occur); a flat table loses this.
- **Multiple ontologies were conflated.** Failure properties, mitigation properties, context modulators, and supervisor self-knowledge are different kinds of things, and a single table forced them into one shape.
- **Failures were treated as points.** Failures are trajectories. The natural representation is a process diagram, not a row.

The replacement vocabulary:
- **Risk-profile dimensions** for failure-intrinsic properties (the four above)
- **Mitigation-system properties** for response-side characteristics
- **Context modulators** for environmental factors
- **Epistemic markers** for supervisor self-knowledge

Three different vocabularies, three different ontologies, kept separate in framework documentation.

The closest mature engineering analog is **bow-tie analysis** (process-safety / risk engineering): causes → event → consequences with barriers at junctions. Bayesian networks and STAMP are more sophisticated alternatives if and when the framework warrants the heavier machinery.

## Hauntings

These are not "things we will resolve in v2." These are presences that follow the schema and must be kept in awareness even when the surface analysis looks clean. Each one represents a way the framework could be silently misleading at the moment it appears most useful.

### H1 — Independence assumption between the four dimensions

The product `P(occur) × P(undetected) × P(unrecovered) × Impact` assumes the four factors are independent. They aren't. A failure that's hard to detect is often also hard to recover from once detected (you've lost time; damage has spread). A failure with high Impact often induces psychological pressure that degrades P(undetected) — supervisors panic-search and miss things. The schema's arithmetic is correct only for the independence case. When dimensions correlate positively, the schema *understates* risk.

**What this haunts:** any autonomy decision that looks acceptable because the product of individually-moderate factors comes out small. Correlations between factors can mean the realized risk is much higher than the product suggests.

**What would let us discharge this:** a joint-distribution model. Not yet warranted by data; flagged for revisit when we have enough instances to estimate correlations.

### H2 — Trajectory model assumes discrete events

The bow-tie / "trajectory through stages" pattern handles discrete-event failures well: something triggers, the event happens, it is or isn't detected, recovery is attempted, residual damage remains. F1, F3, F5 fit this pattern cleanly.

F4 doesn't. Silent drift is not a discrete event — *the failure is the slow accumulation itself*. There's no "occurrence" instant to detect; there's a gradient. Asking "what is P(undetected) for drift?" applies a discrete-event question to a continuous-process failure.

**What this haunts:** any analysis of slow-burn failures using the discrete schema. F4 was caught not by detection mechanisms keyed to events but by the user noticing pattern absence. The schema as written would have given F4 a clean bow-tie diagram that mostly hides what made it dangerous.

**What would let us discharge this:** a second pattern for continuous failures — something like trend-monitoring / control-chart analysis where the "event" is "trend exceeds bound" rather than a discrete trigger. Maybe two patterns is the right number of failure-shape archetypes. Maybe more. Don't know yet.

### H3 — N=1 ontology basis

The schema was extracted from one project's failure log (agentic-mail v0). It has not been tested against f1-modelling failures, agentic-ops failures (which are partial — P5 only), or any non-dev-tools failure space (ML pipelines, infrastructure, web apps, data systems). The same critique I applied to "private → public → deployed → at-scale" as a project trajectory applies to this schema as a risk taxonomy: it might encode the idiosyncrasies of dev-tools failure modes as if they were universal.

**What this haunts:** any cross-project claim about autonomy decisions. The schema's confidence on a new project type should start low and grow with evidence, not be inherited from agentic-mail's confidence level.

**What would let us discharge this:** N=3 across distinct project types with the schema deliberately stress-tested (does the four-dimension decomposition still feel adequate, or do new dimensions appear). Until then, every cross-project application carries a discount on its confidence.

### H4 — Calibration measurement gap

The epistemic ontology names calibration as a meta-property over the schema's outputs, but does not specify *how to measure* it. Calibration is operationally meaningful only when predictions are made, recorded, and later verified. The schema does not yet require predictions to be recorded with the estimates they're based on, so calibration is currently rhetorical — invoked but unmeasured.

**What this haunts:** any confidence claim the supervisor makes about its own performance. Without operationalized calibration tracking, "I'm well-calibrated on this failure type" is unverifiable assertion.

**What would let us discharge this:** every autonomy decision the schema informs should produce a recorded prediction (e.g. "expected escalation density in next phase: X"). Verify after the phase. Maintain a calibration ledger. Six months in, the ledger tells you whether the supervisor's confidence tracks reality.

### H5 — Description vs. decision conflation

The schema describes risk but does not set the autonomy threshold. The dividing line between "this risk profile is acceptable for autonomy level X" and "this risk profile is not" is operator judgment shaped by risk appetite, project stakes, and operator-specific factors. It is tempting to read schema outputs as decisions ("this scored low, therefore expand autonomy"). They aren't. Forgetting this turns the schema from analytical aid into rubber stamp.

**What this haunts:** any moment where the schema produces a clean-looking output and the operator (or the supervisor!) feels the decision is "done." It isn't done. The decision was always operator-side; the schema only fed it.

**What would let us discharge this:** every schema-informed decision must produce a separate decision artifact (`AUTONOMY-DECISION-<date>.md`) that records the operator's threshold reasoning *in addition to* the schema's output. The two artifacts together make the line between description and decision visible.

### H6 — The unknown-unknowns gap

The schema operates on a *known failure registry*. It can characterize, estimate, and inform decisions about failures we have already named. It is silent on failures not yet in the registry — by construction, those failures have no dimensions because we don't know they exist.

This is the deepest haunting because it inverts the schema's apparent strength. The schema is most confident about well-characterized failure modes (where we have N≥several instances). The most dangerous failures, in any new project, are the ones not yet seen — and the schema cannot help us anticipate them.

**What this haunts:** the schema's promise to "inform autonomy expansion." Autonomy expansion is fundamentally about *operating in new conditions*, and new conditions surface new failure modes. The schema as written gives the operator no guidance for the failures their expansion will produce that haven't happened yet.

**What would let us discharge this:** a discovery discipline alongside the analytical schema. Mandatory novelty escalation (when the supervisor encounters anything not in the registry, flag for human review, do not pattern-match into existing categories). Periodic "what classes of failures would we miss?" red-team sessions, conducted by the operator, that explicitly try to imagine modes the registry doesn't cover. Reading the failure literature of adjacent fields (process safety, aviation, medical device certification) to import failure-mode archetypes the project hasn't yet encountered first-hand.

### Meta-haunting — the schema may itself be a category mistake

The schema decomposes "supervisor autonomy decisions" into measurable parts. It is possible that the actual structure of safe AI-agential operation is not a decomposition problem at all — that the right framing is closer to "what kind of relationship between operator and agent does this autonomy level constitute, and is that relationship sound?" That's a different question. The schema as built can't answer it.

**What this haunts:** every confident-looking output of the schema. The framing might be the wrong shape, and the schema would still produce plausible-looking outputs because it's internally consistent. Plausible-looking-from-inside is not evidence of correct framing.

**What would let us discharge this:** sustained operationalization with willingness to discard the schema if it produces decisions that turn out badly even when its outputs looked acceptable. The schema must be *falsifiable in practice*, not just theoretically — and that requires keeping the discipline of recording predictions and post-mortem-ing surprises rigorously enough that the schema's failures can be detected.

## Operationalization plan

To test whether this schema actually helps build better dynamic AI systems rather than just feeling like it does:

1. **Apply the schema** to the autonomy decisions in agentic-mail v0.1.x as those emerge. Record the schema's output, the operator's threshold reasoning, and the actual outcome.

2. **Apply the schema during f1-modelling onboarding.** Use it to characterize the failure modes specific to ML/data work. Note which of the four failure-intrinsic dimensions feel adequate, which feel forced, and whether new dimensions surface.

3. **Maintain the calibration ledger.** Every schema-informed prediction goes in. Every verified outcome closes its prediction. After 90 days, examine: does the schema's confidence track reality?

4. **Conduct a quarterly red-team** specifically targeting the unknown-unknowns haunting: imagine failure modes the registry doesn't cover, log them as hypothetical, watch whether they appear empirically.

5. **At every revision, ask: does the schema still feel adequate?** Resist the urge to tweak it to absorb each new finding into existing dimensions. Some findings should *break* the schema rather than refine it; treat those as more informative than the findings that fit.

The signal that the schema is working: autonomy decisions made with it should fail less often than autonomy decisions made without it. The signal it is failing: the failures that occur are systematically the ones the schema gave clean outputs on. Either signal is informative; only the second one is dangerous if not noticed.

---

This schema is a working hypothesis. It is provisionally useful and certainly incomplete. The hauntings above are not bugs to be fixed in v2 — they are durable features of operating with this kind of analytical aid. Keeping them named and visible is the discipline; resolving them is rarely possible, and pretending to resolve them is worse than leaving them present.
