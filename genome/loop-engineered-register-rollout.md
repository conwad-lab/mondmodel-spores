---
type: genome
origin: mondmodel-prime
created: 2026-07-03T04:15:00Z
confidence: 0.85
observations: 1
observation_note: >
  One full end-to-end rollout (three chained Workflow-orchestrated
  phases, ~6.5M subagent tokens, 0 escalations). Single deployment, but
  with dense per-phase metrics and a deterministic guardrail design that
  should replicate independent of domain specifics.
context:
  domain: orchestration_methodology
  constraints: multi-phase content/data rollout where at least one phase mutates a load-bearing data claim (e.g. price) and another generates gated prose
  environment: Claude Code Workflow orchestration (agent/parallel/pipeline), repo-native gate scripts and SKILL BINARY CRITERIA idioms
provenance:
  local_spore: heritage/loop-engineered-register-rollout.json
  ledger: ledger-0324..ledger-0328
signature: sha256:1b449e8aecc633dcf9ce5bb9a025cf1ac8f367f5326592042b49567e37fc162b
---

# Loop-engineered discipline for a multi-phase register rollout

## Summary

Loop-engineer methodology (5 patterns + 5 guardrails, 'no binary exit condition -> no loop') as the structuring discipline for a full editorial-register rollout. Each phase realized as the matching loop pattern with guardrails in code: canon adoption = evaluator-optimizer (multi-critic review workflow, adversarial verify, fix cycle), data QA = headless while-loop (finite worklist, dry-run gate on anchors, counter-equality exit, read-only + separate apply step), editorial generation = evaluator-optimizer (generator grounded in existing source-of-truth, critic gate on voice contract, fail-closed price rule enforced by code lint).

Applied to a real editorial-register rollout: canon adoption ran as an
evaluator-optimizer loop (multi-critic review workflow, adversarial
verify, fix cycle); a load-bearing data QA pass ran as a headless
while-loop (finite worklist, dry-run anchor gate, counter-equality exit,
strictly separated read-only verification from the apply step); editorial
generation ran as another evaluator-optimizer loop (generator grounded
only in existing verified source-of-truth, critic gate bound to a voice
contract, a fail-closed rule enforced by code lint rather than prose
instruction).

## The genome (replicable procedure)

1. Install taxonomy + guardrail checklist mapped onto EXISTING repo idioms (Workflow-js / gate-scripts / SKILL BINARY CRITERIA) instead of porting the source repo's bash templates
2. Canonize the domain concept in brand-context BEFORE any generation; run a multi-critic review workflow over the canon diff and fix all confirmed findings (11 found incl. the canon's own example violating its own rule)
3. QA the load-bearing data claim (price) with a headless loop: 5-anchor dry-run gate, then full sweep; verdicts CONFIRMED/DRIFT/UNVERIFIABLE computed deterministically in code, never by agent judgment
4. Apply corrections with a deterministic source-quality rule (only primary-source/official-site verdicts mutate; aggregator observations parked) and enforce band-contract consequences (untag hotels that fall below the floor)
5. Generate register editorial grounded ONLY in existing verified editorial (factsUsed recorded), gate through critic, lint the fail-closed rule mechanically (zero price digits where confidence < HIGH)
6. Every irreversible step behind a human checkpoint: canon merge, price apply, Supabase sync each separately operator-gated; drafts live as pending_review report files until approved

## Hard rules (guardrails in code, not prose)

- Binary exit conditions live in code (counter equality, exit codes, grep predicates) - never in prose
- A QA verdict's source TYPE determines mutation rights: official-site = HIGH may mutate; aggregator = MEDIUM parks
- A register's data contract has consequences: correcting a price below the band floor MUST untag ('a persona chip is a promise')
- The critic gate catches the embarrassing class: the canon's own worked example broke the canon's own price rule
- Workflow args may not reach the script sandbox - inline worklists as consts in the script body

## Origin case metrics

Smart-luxury register rollout Mallorca 2026-07-03: 3 workflows (review 15 agents / price-QA 60 agents / editorial 32 agents, ~6.5M subagent tokens), ledger-0324..0328. Outcomes: 11 review findings fixed pre-merge, 60/60 price verdicts, 7 corrections applied + 24 parked by source-quality rule, register 16->14 by band contract, 16/16 editorial drafts passed critic first-cycle, 14 hotels x 4 locales live at /travel/mallorca/best-for/smart-luxury.

- **review findings confirmed**: 11
- **review findings blocking**: 0
- **price verdicts**: 60
- **corrections applied**: 7
- **corrections parked**: 24
- **editorial first cycle pass rate**: 16/16
- **escalations**: 0
- **fix cycles used vs cap**: 0 of 1 per hotel (cap never hit -> caps were generous, keep)

## Calibration notes (what the numbers actually mean)

Dry-run anchor gate (3-of-5 usable) passed with exactly enough margin to be meaningful. DRIFT tolerance +/-25% produced 31 DRIFT of which only 8 had mutation-grade sources - the tolerance is fine, the source-quality gate is what carries the integrity load. Editorial critic passed everything first cycle: either generator prompts were well-bound by canon (likely - canon was written first) or the critic is too lenient (spot-check next run).

## Why this matters beyond the trigger rollout

The core discipline is "no binary exit condition -> no loop": every
autonomous phase must have its exit condition computable in code (counter
equality, exit codes, grep predicates), never left to agent judgment
prose. Data mutation rights are gated by SOURCE TYPE, not by confidence
language -- official-site verdicts may mutate, aggregator-sourced
verdicts park. Every irreversible step (canon merge, price apply,
downstream sync) sits behind its own separate human checkpoint rather
than one blanket approval at the end. And the critic gate exists to
catch the embarrassing class of error: in this rollout, the canon
document's OWN worked example violated the canon's own price rule -- a
whole-system review is what caught something a single-document read
would miss.

## Transfer surface

- Any multi-phase content/data rollout with at least one price/fact-bearing
  data QA pass and one generation pass gated by a voice/quality contract
- Any pipeline where "trust the agent's judgment on when to stop" is being
  considered as the loop's exit condition -- this is the pattern that
  replaces it with a deterministic one
- Any register/taxonomy rollout with band-contract consequences (a tag or
  tier that must be revoked when the underlying data falls out of range)
