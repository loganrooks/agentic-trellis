# Open questions

> Named unknowns. The discipline is to make ignorance explicit rather than let it lurk as silent assumption. Each question has a category indicating what kind of resolution would close it.

Categories:
- **Empirical** — needs evidence from operating the plugin / observing projects
- **Design** — needs decision, not data; deferred until forced
- **Technical** — needs research into Claude Code internals, plugin format, integration mechanics
- **Research** — needs investigation that could plausibly take weeks or months; tracked here so it doesn't get lost
- **Strategic** — needs operator judgment about goals and priorities

## Consultant presentation

**Q1.** How does the consultant present itself to the user? A single slash command? A skill that's invoked conversationally? A series of subagents that coordinate? Something else? (Design, deferred to Phase 2.)

**Q2.** Should the consultant be one skill or several? Onboarding, integration evaluation, autonomy decision review, signal collection, re-onboarding might be separate operations or facets of one. Trade-off: discoverability (separate skills are findable) vs. coherence (one skill carries the consultant pattern uniformly). (Design.)

**Q3.** When the consultant asks discovery questions, does it pause for user input mid-skill, or does it produce a question doc the user fills and re-invokes? The former is more fluid; the latter is more auditable. (Design, related to Q1.)

## Floor guardrails

**Q4.** What is the minimum viable floor guardrail set? The conversation identified ~7 primitives (branch protection, agent-identity-aware review, cost budgets, thrash detection, spec/ADR immutability, secret scan, pre-commit hygiene). Are all needed before Phase 2 ships? Or is there a smaller "true floor" plus optional additions? (Design + Empirical.)

**Q5.** How is agent identity actually distinguishable from human identity in GitHub branch protection? Reviews come from user accounts, not agent identities; bot accounts can review but Codex / Claude operating via user credentials look identical to humans. What's the technical mechanism for "non-author review where non-author means non-agent"? (Technical.)

**Q6.** Cost budgets and kill switches: what is the actual implementation? Codex budget? Anthropic API budget tracking? A wrapper script that counts? Per-session vs. per-project? (Technical.)

**Q7.** Thrash detection: how is it implemented as a CI check? Looking at commit patterns over a time window? Counting fix/revert ratio? Heuristics will be wrong; what's the right baseline? (Empirical + Design.)

## Project-type overlays

**Q8.** What project types deserve dedicated overlays before public release? Developer tools (we have this), ML/data (f1-modelling will inform), infrastructure/ops (agentic-ops partial). What else? Public web apps? Research code? CLI tools? (Strategic + Empirical.)

**Q9.** How does an overlay actually compose with the floor? Layered files (overlay adds files, doesn't replace)? Patch-style overlay? Templated parameters? The terraform/kustomize analog has multiple resolution strategies. Which fits Claude Code plugin conventions? (Technical + Design.)

## Schema operationalization

**Q10.** How is the calibration ledger structured? One file per prediction? Aggregate ledger? What fields make verification tractable? See `HAUNTINGS-RESPONSE.md` H4 — this needs concrete shape before Phase 2 closes. (Design.)

**Q11.** How is "coupling severity" between failure dimensions actually annotated and used? See H1 — qualitative markers (none / mild / strong) were proposed but the operational use of those markers is unspecified. (Design.)

**Q12.** What does the continuous-drift failure pattern look like in artifact form, alongside the discrete-event bow-tie pattern? See H2 — control-chart was suggested as the analog but the concrete schema isn't drafted. (Design.)

**Q13.** What does novelty-escalation look like in practice? When the supervisor encounters a failure mode not in the registry, what is the actual escalation mechanism? See H6. (Design + Technical.)

## Multi-machine / multi-user

**Q14.** All artifacts produced so far assume single-machine, single-user operation. How does this generalize to a team where multiple humans operate the same project with the plugin installed? Whose feedback memory wins? How do supervisor handoffs work across humans? (Design, deferred until needed.)

**Q15.** Where does delegation scope live when supervisors change across humans (e.g. user A's Claude reviewed PR #3; user B's Claude reviews PR #4)? Per-project repo file works (see DESIGN-NOTES.md "setup-as-code"), but the consultant needs to be opinionated about it. (Design.)

## Adversarial threat model

**Q16.** What is the threat model for the plugin itself? It operates with elevated authority over user repos. Could a malicious project trick it into bad actions? Could prompt injection in PR content cause cross-repo harm? See `seeds/SUPERVISOR-FAILURES-2026-05-12.md` for the accidental failures already observed; the adversarial set is uninvestigated. (Research + Technical.)

**Q17.** When a project goes public-facing (visibility dimension changes), what guardrails need to come online specifically against adversarial PRs / issues? Rate limits, content filtering, sandbox isolation? (Design + Research.)

## Versioning

**Q18.** How are projects pinned to plugin versions? The "ONBOARDING-DECISION.md records plugin version" idea was sketched but the migration path between plugin versions is unspecified. Re-onboarding to a newer plugin version: regenerate from scratch, or apply a delta? (Design.)

**Q19.** What's the deprecation policy for plugin features? Some early decisions will need to be reversed as evidence accumulates. How are operator projects affected when the plugin's recommended floor changes? (Design + Strategic.)

## Claude Code plugin specifics

**Q20.** What is the current canonical structure for a Claude Code plugin? Skills, slash commands, hooks, agents — which of these are the right vehicle for which capabilities? Discovery is needed before scaffolding can begin. (Technical, near-term.)

**Q21.** How are plugin-installed hooks distinguished from user-installed hooks? The agentic-mail project's hook system fired effectively in the supervisor failure log (see F1 and F5 in seeds). Can plugin-distributed hooks have the same authority? (Technical.)

**Q22.** How is plugin state persisted between sessions? Memory files, project files, both? The setup-as-code principle says project files; the consultant's own state (catalog of known projects, calibration ledger pointer) might need machine-local state. (Technical + Design.)

## Catalog maintenance

**Q23.** The consultant has to know what's available to recommend (skills, MCP servers, GitHub Actions, CI primitives). That catalog is its own discipline. Should it be community-extensible? If yes, what's the contribution / curation process? If no, how does it stay current? (Strategic + Design, deferred but flagged.)

**Q24.** When the plugin recommends installing a specific external tool (e.g. CodeRabbit, a particular MCP server), what's the trust model? Audit the tool? Trust by reputation? Plugin user assumes responsibility? (Design + Strategic.)

## Self-reference

**Q25.** The plugin asks operator projects to maintain failure logs, signal collections, escalation records. Does the plugin itself maintain these — for the plugin's own development? It should, by dogfooding logic. What does that look like in this repo? (Design, near-term.)

**Q26.** When the plugin's own decisions about its development drift (e.g. spec amendments to the role interfaces), how is the drift tracked? Erratum bundles like agentic-mail uses? Inline in the spec file with revision markers? (Design.)

## Meta

**Q27.** Is the "AI agential systems consultant" framing right at all? See the meta-haunting in `seeds/SUPERVISOR-AUTONOMY-SCHEMA-2026-05-12.md`. Alternative framings exist; none have been seriously tried. When would we revisit the framing? (Research + Strategic, durable open question.)

**Q28.** How do we know when to stop iterating on the plugin and call v1.0? The honest answer is "when projects operating under it fail less often than they would otherwise." But measuring that requires Phase 6 infrastructure plus comparison against a counterfactual that's hard to construct. (Strategic, deferred until late.)
