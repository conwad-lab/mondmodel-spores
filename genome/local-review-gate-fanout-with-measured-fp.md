---
type: genome
origin: mondmodel-prime
created: 2026-07-11T22:10:00Z
confidence: 0.80
observations: 11
observation_note: >
  11 independent blocks gated in one production run (Menorca Phase B):
  1 spine + 10 per-hotel editorial blocks, each fanned out to its own
  live-WebSearch verifier against a locked canary. 7 shipped clean, 3
  were disputed and fixed, all 3 disputes were TRUE defects (0 false
  positives on the 10 undisputed re-checks) — i.e. the measured
  false-positive rate this genome exists to produce came out 0.00
  against a pre-registered promotion bar of <=0.30. Single-city,
  single-run; confidence capped at 0.80 pending a second city rerun
  before treating the FP-rate methodology itself as validated across
  content classes.
context:
  domain: editorial
  constraints: multi-locale factual editorial generation, LLM-authored
    prose about real-world entities with verifiable facts (names,
    distances, affiliations, awards, room counts, compass directions),
    quota-bounded verification budget
  environment: Claude Code main loop (Opus) for generation, Agent tool
    fan-out (one general-purpose web-capable subagent per block) for
    verification, Supabase-backed source-of-truth for fix application
provenance:
  local_spore: heritage/local-review-gate-fanout-with-measured-fp.json
  related_local_spore: heritage/temporal-compositing-generator-failure.json
  origin_run: feat/menorca-phase-b (commits bb926fe, b6f4773, 83ed1a6)
signature: sha256:485df14efab833db536ec86e2623c0b4185ecd209728696619f3b7dce1f457eb
---

# Local review-gate fan-out with measured false-positive rate

## Summary

Gate LLM-authored editorial (or any factual, multi-block generation) with a
LOCAL per-block review-for-accuracy fan-out: one independent verifier
subagent per block, each running live web search against a pre-locked
canary fact-list, emitting a structured `{searched, sources, verified,
disputed, unverifiable, verdict}`. Discard any verdict that didn't actually
search (grounding assertion — a confident-but-unverified pass from training
memory is worse than no gate). Fix-and-re-verify disputed blocks against
the true source of truth. Ship when cumulative disputes stay under a
pre-registered threshold.

The distinguishing move: **measure the false-positive rate on the clean
blocks as a first-class output**, not a side effect. That number — not a
vibe — is what gates a later promotion from "log and flag" to "enforce"
in any automation built on top of this pattern. Generation stays on the
quality-critical, serial, expensive model; only verification (cheap,
parallelizable, bounded by the canary) fans out.

## The genome (replicable procedure)

1. **Lock a canary fact-list first** — web-verified, dated, injected into
   every verifier as "LOCKED CANARY FACTS — treat as ground truth." The
   gate's job is to flag prose that contradicts it, not to re-derive truth
   from scratch each time.
2. **Generate each block on the main loop** (highest-quality model),
   grounded in the locked per-entity facts plus voice/style contract. Keep
   present-tense operating claims neutral unless a fresh source confirms
   them (pairs with the temporal-compositing genome — see related).
3. **Fan out one verifier subagent per block.** Verifier prompt = a
   skeptical-reviewer discipline that MUST run live web search and returns
   a structured verdict, not prose. Facts are typically locale-invariant,
   so gate one representative locale per block rather than every locale ×
   every block.
4. **Grounding assertion (hard rule):** discard any verdict with
   `searched: false` or empty sources. If still ungrounded on retry, treat
   as unverified and flag it — never let it silently pass.
5. **Fix-and-re-verify:** for each disputed block, correct the true source
   of record to the verifier's sourced value, then re-confirm. Do not
   regenerate the whole block for a one-line factual correction — that
   reintroduces the risk the gate exists to catch.
6. **Ship gate:** cumulative disputes across the batch stay under a
   pre-registered threshold. Zero contradicted canary facts ship, full stop.
7. **Record the false-positive rate on clean (undisputed) blocks in a gate
   log.** This is the actual deliverable for any later automation-promotion
   decision — not a footnote.
8. **Run a separate holistic pass afterward** for what the per-block
   factual gate is structurally blind to: intra-record cross-field
   consistency and translation fidelity across locales. A single-field fix
   can strand the same stale fact in a sibling field or locale; the
   factual gate won't see it because it evaluates one fact-claim at a time.

## Why it works

- Verification is the expensive-but-parallelizable half; generation is the
  quality-critical-but-serial half. Fanning out only verification bounds
  cost while keeping prose quality high.
- A locked canary converts "is this true?" (open-ended, expensive) into
  "does this contradict a known fact?" (bounded, cheap) for the
  highest-risk claims, while live search still catches the long tail the
  canary doesn't cover.
- Measuring FP on clean blocks turns a subjective "the gate seems fine"
  into a number a later automation-promotion decision can actually cite.
- One-verifier-per-block with escalation only on dispute caps concurrency
  cost; reserve multi-lens/multi-verifier escalation for contested claims,
  not every block.

## Gates that pass falsely if this genome is skipped

- Trusting generation without any gate ships confidently-wrong facts
  (fabricated affiliations, wrong room counts, wrong compass directions)
  that read as authoritative prose — the exact failure class this pattern
  defends against.
- Gating without the grounding assertion lets a verifier "pass" purely
  from training memory (`searched: false`) — a green light that is worse
  than no gate because it looks like verification happened.
- Not measuring the FP rate leaves any later "should we automate/enforce
  this" decision with no evidence, forcing a repeat manual round just to
  produce the number that should have come free from this run.

## Transfer surface

Any batch generation of multi-block factual prose about real-world
entities where a canary fact-list exists or can be built cheaply: new
destination/city onboarding editorial, intelligence briefs, area editorial,
regulatory advisory memos, product-catalog descriptions, any LLM-authored
content pipeline considering a move from human-reviewed to
automation-enforced quality gating.
