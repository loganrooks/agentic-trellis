# Responding to the hauntings

> The autonomy schema in `seeds/SUPERVISOR-AUTONOMY-SCHEMA-2026-05-12.md` names six hauntings plus a meta-haunting. Naming them is the first discipline. Acting in their presence is the second. This document is the second discipline: for each haunting, the domain of validity, when we operate outside it, mitigations when we do, whether the assumption is necessary, and a recommendation.

The framing distinction matters: hauntings are not bugs to fix in v2. They are presences that follow the framework around and must be kept in awareness even when surface analysis looks clean. Some discharge cheaply once operationalized; some are durable features of operating under finite resources.

## H1 — Independence assumption between failure dimensions

The schema factors expected residual harm as `P(occur) × P(undetected | occur) × P(unrecovered | detected) × Impact(uncorrected)`, treating the four factors as independent.

**Domain of validity.** When the four factors are causally independent. Holds for failures where triggering conditions don't influence detection environments, detection systems don't share resources with recovery, and impact magnitude doesn't reach back into prior stages. Some failures fit cleanly.

**When we operate outside it.** Most real failures couple. *Stress coupling:* high-impact failures induce panic that degrades detection. *Resource coupling:* detection and recovery share scarce attention. *Temporal coupling:* failures that take longer to detect have worsened by the time recovery starts. F4 in the failure log exhibits at least two of these.

**Mitigations.**
- Treat the product as a *lower bound* on risk, not an estimate. Independent factorization can only understate, not overstate, risk when correlation is positive.
- Annotate each registry entry with a qualitative coupling-severity marker (none / mild / strong). Strong-coupling failures get a conservative threshold even when their independent product looks acceptable.
- For high-stakes decisions, run a specific scenario instead of trusting the product: "if A happens, what does that do to B's effective P(undetected)?" Surface the specific pathway rather than averaging it away.

**Necessity.** Tractability concession. Alternatives — joint distributions, Bayesian networks, full scenario analysis — exist and are mature, but require more data than we have or more interpretive effort per decision than is sustainable. The assumption is the price of operating at all under finite resources.

**Recommendation.** Keep the assumption with mitigations. Standing haunting. Revisit when N=3+ provides enough data to estimate factor correlations.

## H2 — Discrete-event trajectory model

The schema treats failures as trajectories with a clear triggering moment.

**Domain of validity.** Failures with a clear occurrence instant — an action, a commit, a decision. F1 (URL fabrication at comment-send), F3 (escape evaluation at heredoc), F5 (post attempt) all fit. The bow-tie pattern from process-safety engineering works cleanly here.

**When we operate outside it.** Drift failures (F4 is the canonical case). Threshold-accumulation failures. Distributed-cause failures (organizational complacency, slow erosion of discipline). For these, *the failure IS the accumulation* — there's no "occurrence" instant, only a gradient. Asking "what is P(undetected)?" applies an event-shaped question to a gradient-shaped failure.

**Mitigations.**
- Maintain a parallel pattern for continuous failures: control-chart / trend-monitoring rather than event detection. The "event" becomes "trend crosses bound" but the modeling shape is fundamentally different (time series, not trigger).
- At each registry entry, classify upfront: discrete-event or continuous-drift. Some failures need both treatments (slow-drift failure with discrete-event triggers superimposed — a drifting supervisor finally takes a specific bad action).
- For continuous failures, the reconciliation primitive named for F4 (periodic "believed-state vs ground-truth-state" comparison, with the *gap* as monitored quantity) is the right pattern.

**Necessity.** No — adding a second pattern is cheap. The evidence already exists (F4) for a class the current schema handles poorly.

**Recommendation.** Drop the single-pattern assumption now. Add continuous-drift as a sibling pattern. Two failure-shape archetypes is a small extension and probably right.

## H3 — N=1 ontology basis

The schema was extracted from one project's failure log.

**Domain of validity.** For projects sufficiently similar to agentic-mail — developer tools, single-repo, Codex + Claude duo, vendored install pattern, spec-with-ADRs governance.

**When we operate outside it.** Immediately, with f1-modelling onboarding. ML/data work has a different failure landscape — silent metric regression, data-code version skew, train/eval contamination, overfitting to held-out, label drift. None of these map cleanly onto the agentic-mail registry. Same for infrastructure projects (cascading service failures, resource exhaustion races) or any public-facing project (adversarial pressure becomes primary).

**Mitigations.**
- Treat the schema's confidence on a new project type as *zero* until empirical data accumulates. Do not transfer agentic-mail's confidence to new project types.
- Bootstrap project-type-specific registries from adjacent literature: MLOps for ML, SRE for infrastructure, security-engineering for public-facing. The *four dimensions* probably generalize; the *registries* shouldn't be carried across.
- Mark every cross-domain application with an explicit "untested in this domain" flag. Schema outputs still inform decisions, with substantial discount.
- When a new project's failure mode doesn't fit existing categories, that's high-priority signal — possibly the schema needs structural revision, not just registry extension.

**Necessity.** Partial. The four dimensions probably generalize; the registry contents shouldn't be assumed to. Drop the registry-transfer assumption; keep the dimensional structure provisionally and watch for evidence against.

**Recommendation.** Drop registry-transfer explicitly. Build per-project-type registries. Keep dimensional structure as meta-layer over domain-specific frameworks. Per-domain frameworks (MLOps, SRE) are the validated artifacts; the universal schema sits above them as a translation layer.

## H4 — Calibration measurement gap

The schema names calibration as a meta-property over its outputs but does not specify how to measure it.

**Domain of validity.** "After we have a calibration ledger with verified outcomes." We are currently *outside* this domain. Every confidence claim is unmeasured.

**When we operate outside it.** Right now, always. To get inside we have to start.

**Mitigations.**
- Treat unverified confidence as approximately equivalent to no information. "I'm well-calibrated on this failure type" without a ledger is rhetorical.
- Default to external validation for any class of failure where the supervisor lacks calibration evidence — human review, machine check, cross-agent verification.
- Be especially suspicious of *high*-confidence claims absent calibration. Confidence without feedback is the textbook setting for miscalibration upward.

**Necessity.** No — entirely avoidable by starting the ledger. The cost is procedural: every schema-informed prediction needs to be recorded with enough specificity to be verified after, and someone has to do the verification pass.

**Recommendation.** Drop the assumption by starting the ledger immediately. This is the cheapest haunting to discharge. Every autonomy-decision artifact from this point should include schema predictions in verifiable form. Six months in, examine the ledger.

## H5 — Description vs. decision conflation

The schema describes risk; the operator decides on thresholds and final actions. Conflating these turns analytical aid into rubber stamp.

**Domain of validity.** Not actually an assumption embedded in the schema — a usage failure mode. The discipline against conflation has a "domain" wherever it is maintained.

**When we operate outside it.** Whenever the operator (or the supervisor!) reads a clean schema output as "this decision is made." Happens under time pressure, cognitive load, when the schema's analysis looks comprehensive enough to substitute for judgment.

**Mitigations.** The conflation discipline is the mitigation. Maintain a separate decision artifact per autonomy decision that records the operator's threshold reasoning *in addition to* the schema's output.

**Necessity.** Not an assumption to drop — a usage hazard to vigilantly avoid. The discipline against it costs almost nothing once habituated.

**Recommendation.** Encode in tooling. Require an `AUTONOMY-DECISION-*.md` per decision that has the schema output as one section and the operator's threshold reasoning as a separate, mandatory section. The artifact prevents conflation by structural means.

## H6 — Unknown-unknowns

The schema operates on a known failure registry. It is silent on failures not yet observed.

**Domain of validity.** For failure modes already in the registry. The schema can characterize and inform decisions about failures we have seen.

**When we operate outside it.** Constantly, especially during autonomy expansion. Autonomy expansion *is* operating into new conditions, and new conditions surface new failure modes by definition. The schema is weakest precisely when it appears most needed.

**Mitigations.**
- **Novelty escalation discipline.** When the supervisor encounters anything that doesn't pattern-match existing registry entries, escalate to human review rather than force-fitting into the nearest category. Force-fitting is the dangerous failure mode — it makes unknown failures look known.
- **Imagination-based red-team.** Periodic operator-side exercise: imagine failure modes that haven't happened. Add as hypothetical entries with predicted dimension values. Watch for empirical confirmation or contradiction.
- **Adjacent-literature import.** Read failure-mode catalogs from related fields. The *categories* they name suggest gaps in our registry even when specific failures don't map.
- **Conservatism in novel territory.** When operating outside the registry's coverage, default to *lower* autonomy, not higher. The schema's silence about a new condition is information about epistemic position.

**Necessity.** The schema *cannot* assume registry completeness — it's structurally impossible. The question is how to act *given* incompleteness; the mitigations above are the answer. The discipline is making incompleteness explicit and the response to it routine.

**Recommendation.** Treat as permanent haunting. Mitigations are the discipline. The most important is novelty-escalation: pattern-matching new conditions to existing categories is exactly how the schema becomes dangerous.

## Meta-haunting — schema might be a category mistake

The decomposition approach might itself be wrong-shaped. Safe AI-agential operation might not decompose into measurable dimensions; the right framing might be relational, virtue-based, or holistic-narrative.

**Domain of validity.** The decomposition probably captures a significant portion of autonomy-decision-relevant structure for failures we can articulate. It almost certainly misses dimensions that don't decompose well.

**When we operate outside it.** Whenever operator intuition disagrees with schema output, or whenever something "feels off" despite clean schema results. These are signals the framing isn't capturing something. They are evidence about the schema's framing, not noise to be suppressed.

**Mitigations.**
- **Don't be schema-bound.** Operator intuition contradicting schema output is data, not error.
- **Periodically reframe.** Quarterly: ask "what would this autonomy decision look like under a relational frame? a virtue frame? a holistic-narrative frame?" If the answer differs, that's interesting.
- **Cross-frame triangulation for important decisions.** Convergence across frames is evidence; divergence is also evidence — specifically, evidence that no single frame is adequate and the decision should be more conservative than any one of them suggests.

**Necessity.** Some framing is necessary; *this* framing is one bet. Alternatives have costs:
- Pure case-by-case operator judgment: doesn't accumulate learning across decisions
- Pure procedural caution: doesn't scale; might block operation
- Relational/virtue framings: probably capture things decomposition misses but haven't been operationalized for agentic dev contexts
- No autonomy at all: doesn't realize the value of agentic dev

The assumption is "decomposition is a useful working hypothesis." That bet might be wrong, but it's tractable and alternatives aren't obviously better.

**Recommendation.** Keep schema as primary working hypothesis. Maintain quarterly reframing exercise. Treat operator intuition disagreeing with schema output as high-signal evidence.

## Synthesis: looking at the assumptions as a whole

A pattern emerges from per-haunting analysis:

- **H4 and H5** are cheap to drop. They're procedural disciplines, not deep assumptions. Drop immediately — start the calibration ledger, structure decision artifacts to prevent conflation.
- **H2 and H6** are medium effort. Add the continuous-drift pattern alongside discrete-event; institute novelty-escalation as a standing rule. Should do both.
- **H3** is real domain-of-validity work, triggered by f1-modelling onboarding. Bootstrap per-project-type registries when needed; don't transfer the agentic-mail registry blindly.
- **H1** and the **meta-haunting** are durable. They're tractability concessions (H1) and framing bets (meta) we probably can't fully discharge with finite resources. Standing hauntings.

**Most of the hauntings are cheap-to-medium-cost discharges that should happen in operationalization, not be deferred to v2.** Only two are deep enough to live with permanently.

## The no-assumption path

If we made *none* of the hauntings' assumptions:

- **No independence** → joint scenario analysis for every decision. Expensive.
- **No discrete-event assumption** → continuous-time risk modeling. Conceptually heavier.
- **No registry-transfer** → per-domain framework only. Fragments operator view.
- **No calibration-trust** → every confidence claim verified externally before use. Bottleneck-bound.
- **No analytical-decomposition** → pure case study + intuition. Doesn't transfer.

This would be *technically more defensible* and *operationally infeasible*. The schema is the bet that we can navigate autonomy decisions with finite resources and partial information. Pure no-assumption operation requires either infinite resources or refusal to act.

**The honest framing: we make these assumptions because operating at all under finite resources requires assumptions, and the assumptions we make are negotiable but not eliminable. The discipline is choosing consciously, marking, watching for breakdown, revising as evidence accumulates.**

## What changes in our approach

1. **Immediate (cheap drops).** Start the calibration ledger. Structure autonomy decisions as artifacts with separate "schema output" and "operator threshold reasoning" sections. Add novelty-escalation as a standing supervisor rule. Add continuous-drift as a sibling failure pattern.
2. **At f1-modelling onboarding (forced by H3).** Bootstrap an ML-project failure registry separate from agentic-mail's. Note explicitly that schema confidence on ML failures starts at zero. Use MLOps risk-framework literature as seed.
3. **Standing disciplines.** Quarterly reframing exercise (meta-haunting). Coupling-severity annotation per registry entry (H1). Cross-domain consistency review when N=3 is reached.
4. **What we accept we'll live with.** Independence assumption modulo coupling markers, until joint-distribution data is plentiful. The decomposition-as-framing bet, knowing it might be wrong, watching for the signals that it's wrong.
