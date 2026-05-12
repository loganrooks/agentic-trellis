# Roadmap

> Rough phase order for moving from seed-phase to working plugin. Not committed; each phase has explicit entry/exit conditions and any of them might be substantially revised by what the prior phase discovers.

The roadmap exists to make the order of operations visible, not to lock it in. Phase boundaries are decision points, not deadlines.

## Phase 0 — seed phase (current)

**Entry:** design conversation produced enough material to risk being lost to context.
**Exit:** repository exists, seeds are captured, design notes are written, working hypotheses are explicitly marked.

Artifacts:
- `README.md` — vision and seed-phase framing
- `seeds/SUPERVISOR-FAILURES-2026-05-12.md` — empirical failure log
- `seeds/SUPERVISOR-AUTONOMY-SCHEMA-2026-05-12.md` — analytical framework
- `DESIGN-NOTES.md` — distilled conversation material
- `HAUNTINGS-RESPONSE.md` — per-haunting analysis and discharge plan
- `OPEN-QUESTIONS.md` — explicitly named unknowns

## Phase 1 — specification of primitives

Make the conceptual artifacts concrete enough to use.

**Goals.**
- Specify the four role *interfaces* (planner, executor, supervisor, orchestrator) — schema, inputs, outputs, side-effects, handoff format. Interfaces before defaults.
- Specify the floor guardrails as concrete primitives (CI checks, hook contracts, branch-protection requirements, cost-cap mechanism).
- Specify the failure registry schema with the four risk dimensions plus the three companion ontologies.
- Specify the `ONBOARDING-DECISION.md` and `AUTONOMY-DECISION-*.md` artifact formats.
- Specify the calibration ledger format.

**Exit criteria.** A second supervisor working on a different project (e.g. f1-modelling) could read the specifications and apply them without needing the conversation context that produced them.

**Anti-goal.** Do not implement default adapters yet. Specifying interfaces and immediately implementing defaults silently turns the defaults into the interfaces.

## Phase 2 — consultant skill (minimum viable)

Build the consultant pattern as a single Claude Code skill that can be invoked on an existing project.

**Goals.**
- Discovery: skill reads existing artifacts, asks discovery questions, classifies risk-exposure dimensions.
- Diagnosis: identifies misalignments between current state and appropriate guardrails for the dimension profile.
- Proposal: outputs a diff (files to add, settings to change) before applying.
- Application: applies with auditable commits.
- Validation: runs a synthetic PR through new guardrails to prove they fire.

**Exit criteria.** Skill can be run on a new project (or a re-onboarding of an existing one) and produces an `ONBOARDING-DECISION.md` plus a reviewable diff.

**Test target.** Re-onboard `agentic-mail` itself using the consultant. The skill's first real test is whether it reproduces the discipline already in place when fed the same project state.

## Phase 3 — N=3 stress test

Apply the consultant to two additional project types.

**Goals.**
- Onboard `f1-modelling` (ML/data project) using the consultant.
- Re-engage `agentic-ops` (infrastructure/process project, currently partial coverage from P5 dogfood).
- Track which dimensions the schema's four-dimension structure handles cleanly vs. which need supplementation.
- Track which failure modes appear that don't fit existing registry categories.

**Exit criteria.** Evidence about which assumptions in the schema generalize and which don't. If the schema needs structural revision, this is when it happens.

**Risk.** The temptation will be to force-fit new project failures into existing dimensions. The discipline is to flag novelty rather than absorb it. See H6 in `HAUNTINGS-RESPONSE.md`.

## Phase 4 — adapter pattern

Implement the interface-first, replaceable-defaults principle concretely.

**Goals.**
- Implement at least one comms adapter (agentic-mail) against the comms interface.
- Implement at least one alternative-planner adapter (likely a stub for GSD or another framework) to validate the interface accommodates external tools.
- Define conformance tests per adapter type. Adapters either pass or are explicitly marked non-conforming.

**Exit criteria.** A user can swap one adapter for another and the consultant continues to work, even if some features become unavailable.

## Phase 5 — integration evaluation feature

Build the "integrate framework X" consultant operation as a first-class flow.

**Goals.**
- Given a candidate framework (e.g. GSD), the consultant researches it, maps to interfaces, identifies gaps, proposes a hybrid setup.
- Test case: produce a real integration proposal for GSD.

**Exit criteria.** A user can ask the consultant "should we integrate X?" and get an actionable, auditable answer with rationale.

## Phase 6 — signal infrastructure and calibration

Build the data layer that makes operationalization rigorous.

**Goals.**
- Signal collection in a common format across adapters.
- Calibration ledger that records schema predictions and verifies them after the fact.
- Periodic reflection / deliberation cycle that produces named lessons.

**Exit criteria.** After 90 days of operation, the ledger can answer "does the schema's confidence track reality?" with actual data, not rhetoric.

## Phase 7 — public release

The release that takes the plugin from "experimental for the maintainer" to "installable by others."

**Goals.**
- Documentation for new users (operator-perspective, not author-perspective).
- Install / uninstall paths that meet the discipline asked of operator projects.
- Security review (the plugin operates on user repos with elevated authority; it must be auditable in detail).
- Public roadmap, deprecation policy, release cadence — the very items the plugin asks operator projects to commit to.

**Exit criteria.** A user with Claude + Codex subscriptions can install the plugin, onboard a new project, and operate it for one phase of development without needing the maintainer's direct assistance.

## Non-goals

Things explicitly not on this roadmap:

- **Becoming a replacement for GSD.** GSD-as-optional-adapter is a supported integration; agentic-trellis is not a fork.
- **Provider lock-in.** Default to Claude + Codex by target user definition, but interfaces remain provider-neutral.
- **A single canonical project trajectory.** No private → public → deployed ladder. Dimensions, not stages.
- **Premature framework codification.** Don't extract patterns beyond N=3 until N=3 evidence is in. The N=1 schema is provisional.
- **Hiding the framework's own failures.** The plugin asks operator projects to maintain failure logs; the plugin maintains its own.

## Re-roadmapping triggers

This roadmap should be revised when:

- Phase 3 (N=3) produces evidence the schema's structure is wrong, not just incomplete.
- The consultant skill (Phase 2) discovers that the discovery/diagnose/propose/apply/measure pattern doesn't survive contact with real projects.
- Claude Code plugin format changes substantially.
- Adjacent tooling (GSD-replacement, MCP ecosystem, etc.) makes parts of the roadmap obsolete or redundant.
- Operator usage reveals a critical need not currently on the roadmap.

The roadmap is itself an artifact subject to the discipline it advocates: revised in commits, not silently overwritten, with rationale attached.
