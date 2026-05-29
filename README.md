# agentic-trellis

> A trellis is a structure that supports growth without dictating its shape.

**Status: seed phase.** This repository is not yet a working Claude Code plugin. It is a collection of design artifacts and analytical frameworks captured during a supervisor session on a related project (`agentic-mail`, currently private), intended to seed the development of a plugin that doesn't yet exist. Working hypotheses throughout; nothing here is validated at scale.

_Part of the `agentic-*` family — see the `agentic-ecosystem` repo
(`ECOSYSTEM.md`) for what this repo owns and how it composes with its
siblings._

## What this aims to be

A consultant for AI-agential development workflows — specifically for operators using **Claude as planner/orchestrator/supervisor** and **Codex as executor**. The consultant pattern is borrowed from industrial engineering: assess the situation, diagnose the bottlenecks, propose interventions calibrated to actual risk, implement with auditable diffs, measure outcomes, iterate.

Concretely, it is intended to provide:

- **Onboarding and re-onboarding.** Discovery-driven setup that produces an auditable `ONBOARDING-DECISION.md` recording dimension assessments, chosen overlays, and trigger conditions for re-onboarding. Re-onboarding when project risk dimensions change (visibility, audience size, operational mode, AI authority level, etc.) — not when arbitrary "stages" are crossed.
- **Floor guardrails for AI-agential development.** Primitives most CI/CD practice doesn't provide because it assumes humans push code: agent-identity-aware required review, rate-limited PR creation, cost budgets with kill switches, thrash detection (fix-fix-fix chains, mass file churn) as a CI check, spec/ADR immutability enforcement.
- **Supervisor role specification.** Project-independent definition of standing duties, per-project authority bounds, handoff protocols, and — critically — mandatory failure logging. The supervisor's positive contributions are visible in PR history; its failures vanish unless deliberately recorded.
- **Failure-mode registry with risk dimensions.** Four failure-intrinsic dimensions (occurrence, undetection, unrecovery, residual impact), separated from mitigation-system properties, operating-context modulators, and the supervisor's own epistemic state.
- **Autonomy decision support.** Schema-informed analysis paired with explicit operator-side threshold artifacts. The schema describes risk; the operator decides.
- **Integration evaluation.** When asked "can we integrate X" (GSD, a CI primitive, an MCP server, etc.), the consultant should research, map X's outputs to the plugin's interfaces, identify gaps, and propose a hybrid setup as a diff — not say yes or no.

## What this is not

- Not a working plugin. The `.claude-plugin/` directory does not exist yet by intent — empty scaffolding would suggest implementation maturity that isn't there.
- Not a framework that prescribes one development trajectory. Earlier drafts of this design assumed a "private-experimental → public-pre-release → public-deployed → at-scale" progression. That framing was wrong: it encoded Silicon Valley product-trajectory assumptions and silently excluded research code, internal tooling, public-from-day-one work, throwaway experiments, and many other legitimate paths. The design now assesses risk-exposure dimensions independently.
- Not a substitute for operator judgment. The schema describes risk; threshold-setting and final decisions are operator-side. Conflating the two is a named haunting (see `seeds/SUPERVISOR-AUTONOMY-SCHEMA-2026-05-12.md` H5).
- Not validated. Derived from N=1.5 projects (agentic-mail in full, agentic-ops partially via P5 dogfood). Cross-project generalization is an open question, not a settled fact.

## Repository layout

```
agentic-trellis/
├── README.md           This file
├── seeds/              Design artifacts captured verbatim from the source session
│   ├── SUPERVISOR-FAILURES-2026-05-12.md       Empirical failure log seeding the registry
│   └── SUPERVISOR-AUTONOMY-SCHEMA-2026-05-12.md   Working analytical framework with named hauntings
├── DESIGN-NOTES.md     Distilled conversation material on the consultant pattern, ontologies, and composability
├── HAUNTINGS-RESPONSE.md   Per-haunting analysis: domain of validity, when we operate outside, mitigations, necessity
├── ROADMAP.md          What we know needs to be built, in rough order
└── OPEN-QUESTIONS.md   What we don't yet know; explicit unknowns
```

## How this came about

During the supervisor session that closed out agentic-mail v0 (2026-05-11 → 2026-05-12), a design conversation surfaced the question of whether the auto-execution workflow, supervisor role pattern, and failure-discipline conventions developed for that project might be formalizable as a Claude Code plugin. The conversation produced two artifacts (the failure log and the autonomy schema) and a longer set of design notes that risked being lost to context.

Rather than wait for the plugin design to mature, the decision was to capture the artifacts now, mark them as seeds, and let the actual plugin development happen in this repo's branches as the design is operationalized across additional projects (currently agentic-ops follow-up work and f1-modelling onboarding).

The N=1.5 caveat is important: the same critique the failure log makes of pattern-matching new conditions to existing categories (haunting H6, unknown-unknowns) applies to the schema's own confidence on project types other than developer tooling. Cross-project use should start from a discount, not a transfer.

## License

MIT. See `LICENSE`.

## Provenance

Seed artifacts derive from a supervisor session on the `agentic-mail` project (currently a private repository). The session and its escalation/checkpoint history are gitignored in that project's `.planning/auto-execution/` directory; the documents copied into `seeds/` here are the externally-shareable subset, scrubbed of project-specific path leakage where applicable (none was found in these two files, but the discipline is named explicitly).

This repository's design is also informed by — and explicitly distinct from — GSD Reflect (`gsdr`), a Claude Code planning/execution framework whose roadmapping and spike-integration limitations motivated some of the architectural choices here. GSD-as-optional-adapter is one of the integration patterns the consultant is intended to support; agentic-trellis is not a fork or replacement. (GSD project URL omitted pending verification rather than fabricated — see haunting F1 in `seeds/SUPERVISOR-FAILURES-2026-05-12.md` for why this caveat exists.)
