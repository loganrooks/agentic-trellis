# Design notes

> Distilled from the design conversation that produced the seed artifacts (2026-05-12). These notes capture the conceptual moves and architectural decisions that risked being lost to conversation context. They are not specification — they are working positions, each one revisable, each one with its own provenance traceable to a specific moment in the source session.

## Target user and scope

Operators who hold **both Claude and Codex subscriptions** and want to use them in complementary roles: Claude as planner / orchestrator / supervisor, Codex as executor. The plugin's value proposition exists at this specific intersection; it is not a general-purpose framework for AI-assisted development.

The plugin's scope is **product development workflows** in the broad sense: any project where artifacts are produced, decisions accumulate, and the gap between intent and outcome is non-trivial. It is not restricted to public-facing or deployed projects; private experimental work that uses AI-agential development has its own risk profile and benefits from the same disciplines.

## The consultant pattern

The unifying framing is **AI-agential-systems consultant, modeled on industrial engineering.** Industrial engineers optimize systems for productivity, quality, and safety; they don't impose one factory layout. They have a discipline (operations research, lean, theory of constraints, six sigma) that they apply contextually.

The consultant pattern has five stages, each producing an auditable artifact:

1. **Discovery** — read existing project artifacts (repo, prior commits, planning docs), ask discovery questions, classify project state along risk-exposure dimensions. Output: `ONBOARDING-DECISION.md` recording dimension values *as user-provided answers*, not as consultant assumptions.
2. **Diagnosis** — name where the project's current state is misaligned with appropriate guardrails, where bottlenecks exist, where AI-agential-dev failure modes are under-addressed. Output: a measurable diagnosis with predicted intervention effects.
3. **Proposal** — overlays, role authority bounds, signal collection cadence, CI guardrails, as a reviewable diff. Output: a `terraform-plan`-like preview the operator approves before anything is applied.
4. **Application** — apply with auditable diffs, commit with `[onboarding]` prefix, write decision provenance into the project repo itself (not into per-user memory files).
5. **Measurement and iteration** — instrument for signal collection, watch for predicted effects, re-onboard when dimension values change.

The consultant's *reasoning* is the differentiated value. Everything else (templates, overlays, adapters) is composable plumbing.

## Setup-as-code, not setup-as-process

The plugin's outputs are checked into the project repo, not buried in user-machine memory. Three motivations:

1. **Auditability across machines and contributors.** A supervisor authority file in `~/.claude/` is machine-local; a `.supervisor/AUTHORITY.md` in the project repo is visible to everyone who clones.
2. **Re-onboarding produces diffs.** When dimension values change, re-running the consultant produces a delta against the current setup. The operator reviews; the change is committed as a normal PR.
3. **The framework dogfoods its own discipline.** The consultant is itself an artifact subject to spec immutability + erratum bundles + signal collection + deliberation cycles. Everything the plugin asks of operator projects, it asks of itself.

The closest existing analog is `terraform plan` / `terraform apply` — declarative configuration with a preview step. Composable templates with explicit overlay points (think `kustomize` rather than monolithic frameworks) keep projects from getting locked into the framework's current shape.

## Interfaces before defaults

The plugin defines role *interfaces* before implementing role *defaults*:

- **Planner interface** — what every planner adapter must produce (PLAN.md schema, task breakdown format, phase structure, dependency graph). Default implementation may be built-in; alternatives (e.g. GSD-as-planner) are pluggable.
- **Executor interface** — what every executor adapter consumes (PLAN.md → task execution, commit discipline, escalation triggers, state reporting). Default: Codex `/goal`. Alternatives possible.
- **Supervisor interface** — what every supervisor implements (monitoring duties, push-back authority bounds, failure logging, handoff protocol). Default: Claude-with-feedback-memory. Provider-neutral by design — the interface should not lock to Anthropic.
- **Review adapter** — bot reviewers (CodeRabbit, Codex bot review, custom). The supervisor consumes review output, doesn't generate it directly.
- **Communication adapter** — cross-agent messaging. Default: agentic-mail protocol (currently in a private repository). Alternatives: MCP servers, shared-filesystem disciplines, API call patterns.
- **Cost / observability adapter** — budget tracking, kill switches, signal export.

The hard rule: **define the interface before implementing the default**, or the default silently becomes the interface and adapters can't be swapped. Each adapter has conformance tests; passing tests is the criterion for inclusion, not a vendor blessing.

## Risk-exposure dimensions, not lifecycle stages

An earlier draft proposed a four-stage lifecycle: private-experimental → public-pre-release → public-deployed → public-at-scale. This was wrong. It encoded Silicon Valley product-trajectory assumptions and silently excluded:

- Research code that stays private and dies after producing a paper
- Internal tooling that's private forever
- Public-from-day-one open-source libraries
- Public-but-never-deployed specs and libraries used by others
- Deployed-but-never-public internal systems
- Throwaway experiments
- Forks that exist briefly to test a hypothesis
- Projects that go *backwards* (re-privatize for security, sunset from deployed to archived)
- Projects that pivot so completely that v2 has nothing to do with v0

The replacement: **risk-exposure dimensions, assessed independently**, with guardrails attached to dimension *values* (not to stage labels). Candidate dimensions:

1. **Blast radius** — who's affected when something breaks (only me / collaborators / users / external systems / safety-critical)
2. **Reversibility** — trivial undo / inconvenient / expensive / irreversible
3. **Visibility** — separately: visibility of the project, and visibility of its failures (these are not the same)
4. **AI authority** — what the AI can do without per-action human review
5. **Operational mode** — not-running / on-demand / continuously-running / production-critical
6. **Velocity expectation** — stable / iterative / experimental / spike

Stage transitions become "specific dimension changes" rather than "moved to next stage." A project can:
- Become more visible without becoming more deployed
- Become more deployed without changing visibility
- Increase AI authority while everything else stays put
- Archive (operational mode → not-running, but new guardrails for archival discipline appear)
- Re-privatize, pivot, fork, sunset

Guardrails attach to dimension values regardless of overall project shape. This composes cleanly and excludes no legitimate project trajectory.

## Composability with external frameworks

The plugin is designed so that integrating a framework like GSD (or any future tool) is a first-class operation, not a special case. When asked "can we integrate X?", the consultant:

1. **Detects or installs** X (with permission)
2. **Maps** X's outputs to the plugin's role interfaces
3. **Identifies gaps** — what X does well, what it lacks, where the plugin should supplement vs. substitute
4. **Proposes a hybrid setup** with the integration as a reviewable diff
5. **User reviews and applies**

For GSD specifically, this might look like: use GSD's plan-phase / collect-signals / reflect; substitute Codex `/goal` for execute; have the plugin own roadmap, spike track, and supervisor authority. The plugin's job is to make integration *tractable*, not to be a closed system.

## Failure logging as mandatory artifact

The supervisor's positive contributions are visible in PR history and merge records. Its failures vanish unless deliberately recorded. This asymmetry is itself a failure mode — it biases self-evaluation toward "what works" and hides "what fails."

The failure log (`seeds/SUPERVISOR-FAILURES-2026-05-12.md`) is the first instance of this discipline applied. The plugin's standing rule for any project under its supervision will be: **failure-log creation is a mandatory artifact of every supervisor session**, not an optional discipline.

Failures must be tiered by operational/material consequence:
- **Tier 1**: did happen with material consequence
- **Tier 2**: would have happened, averted by machine guard
- **Tier 3**: averted by existing supervisor discipline before consequence
- **Out of scope**: discussion-time framing missteps in deliberation (different artifact, different stakes)

Conflating tiers degrades the signal the log is supposed to preserve.

## Four ontologies

Risk analysis for AI-agential development requires keeping four ontologies separate:

1. **Failure ontology** — intrinsic properties of failures: occurrence probability, undetection probability, unrecovery probability, residual impact. Conditionally chained, not independent.
2. **Mitigation-system ontology** — guards, hooks, disciplines, recovery procedures, with properties like coverage, cost, robustness, auditability.
3. **Operating-context ontology** — environmental modulators: adversarial pressure, regulatory regime, audience composition, time sensitivity, multi-tenancy, accountability assignment.
4. **Epistemic ontology** — supervisor's knowledge of itself and the failure registry: calibration, registry coverage, recency, provenance.

Mixing these into a flat schema (the original "column table" framing) loses crucial structure. The autonomy decision lives at their intersection.

## Hauntings as first-class

Working hypotheses have limitations that follow them around even when the surface analysis looks clean. These are **hauntings**, not caveats — they persist whether attended to or not, and resolving them is often impossible or premature. The discipline is to keep them named and visible.

Six named hauntings on the autonomy schema (full analysis in `HAUNTINGS-RESPONSE.md`):

- **H1** Independence assumption between failure dimensions (the product factorization is wrong when factors correlate, and they do)
- **H2** Discrete-event trajectory model assumes occurrence is a moment (drift failures like F4 don't fit)
- **H3** N=1 ontology basis — extracted from one project, not yet tested across project types
- **H4** Calibration measurement gap — calibration is named but unmeasured; currently rhetorical
- **H5** Description vs. decision conflation — schema describes risk; operator decides; forgetting this turns analytical aid into rubber stamp
- **H6** Unknown-unknowns — schema operates on a known registry; failures not in it are invisible by construction

Plus the **meta-haunting**: the decomposition approach might itself be a category mistake. Other framings (relational, virtue-based, holistic-narrative) may capture things the decomposition misses.

The mature engineering analog for this kind of structured-but-incomplete risk analysis is **bow-tie analysis** from process safety: causes funnel to event on the left, consequences fan out on the right, barriers at junctions. The schema as currently written follows this pattern for discrete-event failures; H2 names that drift failures need a different pattern.

## Distinct from GSD

GSD (`gsdr`) provided the conventions that agentic-mail's auto-execution structure resembled but did not use. The conscious distance:

- **Roadmap layer.** GSD lacks a strong cross-milestone roadmap with version intent, deprecation policy, release cadence. agentic-trellis is intended to provide this as a top layer above milestones.
- **Spike integration.** GSD has a `spike` skill but the integration into planning is weak. agentic-trellis intends spikes as a parallel track to phases — bounded experiments with decision artifacts that feed planning without requiring a full replan.
- **Agility.** GSD is plan-then-execute oriented. agentic-trellis intends explicit replan and deviation lanes, with deviations as first-class signals rather than exceptions.
- **Maintenance.** GSD is unmaintained at the time of this design. The lessons it surfaced are useful; depending on it operationally is not.

agentic-trellis is not a fork. GSD-as-optional-adapter is a supported integration pattern. The two projects can coexist; the consultant can recommend using GSD for the parts it does well.

## Provenance discipline

Every assertion in this document is provisional. Specific claims trace to specific moments in the source session (the supervisor conversation on 2026-05-11 / 12). When operationalization produces evidence against any of these positions, revisions should be tracked in this repo's commit history, not silently overwritten. The discipline that produced these notes — spec immutability with erratum bundles for governance documents — is the same discipline agentic-trellis asks of operator projects, and it applies recursively to its own framework documentation.
