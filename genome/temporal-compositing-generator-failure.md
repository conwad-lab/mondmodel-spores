---
type: genome
origin: mondmodel-prime
created: 2026-07-11T00:00:00Z
confidence: 0.85
observations: 11
observation_note: >
  11 hard contradictions + 2 staleness flags surfaced by a two-agent
  parallel WebSearch/WebFetch review-for-accuracy pass across one
  generation bundle (spine + 4 intel briefs + 7-area editorial x4
  locales + 12-month x7-area tips). Confidence reflects a single
  well-instrumented incident with a clear decision-tree outcome
  (revert, not patch), not repeated independent confirmations across
  different generation surfaces.
context:
  domain: editorial
  constraints: LLM-generated present-tense prose about multi-year-history entities
  environment: Claude Code editorial generation skills (generate-editorial, generate-destination-content, generate-intelligence-brief, generate-monthly-briefs, generate-area-editorial), no built-in temporal-anchoring requirement in the generator prompt
provenance:
  local_spore: heritage/temporal-compositing-generator-failure.json
  ledger: ledger-0250 (HERITAGE_SPORE_EXTRACTED)
signature: sha256:326eb192b1ed2c4c8c07375706b152d65131ed096c40c4fb1ca67d208941b980
---

# Temporal compositing: correct facts, wrong "now"

## Summary

A distinct failure class from memory-absence hallucination. The generating
model's training memory CONTAINS the relevant facts — they are not
fabricated. The failure is that facts from DIFFERENT time windows get
stitched into a single confident present-tense claim. A venue did host an
event in some past season; a building did have monastic use in some past
century; a statistic did move by some amount at some past date. Each
fragment is true at *some* date. The generator's failure is asserting them
all as true *now* without ever anchoring what "now" means.

Canonical instances from one incident: a castle described with a different
building's convent history ("Almudaina... a sixteenth-century convent" —
Almudaina is a fortress; the convent-to-town-hall building is a separate
structure); a restaurant's past venue asserted as current ("Sublimotion at
Hard Rock Hotel Ibiza" — true for earlier seasons, but the entity relocated
for the season in scope); a statistic's two separate deltas merged into one
event ("aquifer 39%→47% from the October DANA" — the DANA jump was
26%→39%; 39→47 was a distinct November recovery); a spatial impossibility
asserted with confidence ("sightlines including a bay to the northeast" —
blocked by an intervening structure, "feels" plausible but is wrong).

## The genome (replicable procedure)

1. Detect temporal-compositing risk. If the task is 'write present-tense editorial about an entity with a multi-year history' (a venue that's changed operators, a chef with a multi-year career, an architectural project with a long restoration arc, a statistic with a multi-quarter trajectory), tag this as temporal-compositing-prone BEFORE writing prose.
2. Anchor 'now' in the prompt explicitly. The generator prompt must include the system date and the phrase 'every present-tense claim must carry an inline as-of date anchor or cite a source dated within 6 months of <SYSTEM_DATE>.' A green CI on a prompt that does NOT include this clause is a yellow flag at PR review.
3. Forbid bare time-deictics in the generator prompt. The words 'now,' 'currently,' 'this season,' 'today,' 'at the moment' must be paired with a date anchor or replaced with a date-bearing phrase ('as of May 2026,' 'per [DATE] source,' etc.). The forbidden-construction check is mechanical — a regex-grade gate, not a judgment call.
4. Require dated source for every statistic. For prices, percentages, room counts, distances, attendance figures — the generator must cite a source dated within 6 months of system date. The /review-for-accuracy three-month rule for statistics is the upper bound; 6 months is the floor for editorial.
5. Separate 'this happened' from 'this is the case now'. Past events get past-tense framing with their dates. Present-state claims require a fresh source. Mixing them — 'X is the case' where X was true in 2022 — is the canonical failure mode. The generator prompt must distinguish these registers explicitly.
6. Spatial / directional claims require map verification. Compass orientation, sightlines, what-is-visible-from-where are deceptively easy to get wrong because they 'feel' obvious. Sant Bernat → Talamanca is the bellwether — claimed visibility from a southern bastion to a northeastern bay blocked by the cathedral. Require at least one map source for every spatial claim.
7. Treat venue/operator/ownership transitions as a high-risk class. Restaurants relocating, hotels rebranding, restaurants closing-and-replacing-with-new-concept — these are the sharpest temporal-compositing surfaces because the static web page often persists past the operating change. Trust dated press over static-site existence checks for current-state claims (this rule appears in spore #30 step 9 too; here it carries extra weight).
8. When verification reverses a prior 'now' assertion, update the source-of-truth spec in place. §13 #6 RE-REVISION (Estragón May 2026 relocation) was such a reversal. The correction belongs in §13 itself, not in an appendix.
9. When verification surfaces ≥10 prose contradictions in a generation pass with temporal compositing as the dominant class, do NOT patch in place. The generator's prompt — not the model's memory — is the failure surface. Patching individual prose fragments leaves the prompt unchanged; the next regeneration reproduces the failure rate. Revert, update the prompt, regenerate.
10. Distinguish spore #30 (memory absence) from spore #31 (temporal compositing) when running post-mortem. Same outward symptom (wrong factual claims in prose), but different fix surfaces. Spore #30 → add WebSearch tools to skill; spore #31 → add time-stamp clauses to generator prompt. A skill can fail both classes; both fixes can be applied independently.

## Gates that pass falsely (why this survives review)

- npx tsc --noEmit — 0 errors (silent on prose temporality)
- npm run build — passed (validate-city-locales --strict checks shape, not date-anchoring)
- /review-for-accuracy posthoc — could catch temporal compositing only by accident, not by design; the skill's three-month rule for statistics is the closest existing gate but it's reviewer-side, not generator-side
- Voice contract bind — passed (Mondmodel voice doesn't include a 'date-stamped present-tense' clause, so register checks are silent)
- Prohibited-words scan — green (lexical, not temporal)

## Gates that would have caught it

- Generator prompt requiring 'as of <YYYY-MM>' for every present-tense entity claim — would have refused to author 'Sublimotion at Hard Rock' without a 2026-specific source
- Forbidden-construction regex in the SKILL.md ('now', 'currently', 'this season' without paired date anchor) — would have flagged at write time
- Per-statistic dated-source requirement — would have caught aquifer 39→47 telescoping by forcing the generator to cite which event produced which delta
- Spatial/directional claims requiring map verification — would have caught Sant Bernat → Talamanca
- Form A self-invoked /review-for-accuracy with active WebSearch — would have surfaced the 11 contradictions (this is spore #30's gate, applied here as backstop)

## Counterfactuals (what happens without this genome)

- **risk_1**: Publishing temporal-compositing editorial at launch would have shipped 11 verifiable wrong-claim instances across the Ibiza launch corpus. Lower volume than Step 6 (~25 contradictions) but higher reputational damage per error — a 'reopens at Hard Rock' claim for a venue that's relocated to BLESS is the kind of error a single Ibiza journalist's tweet retires the launch's credibility on.
- **risk_2**: Posthoc patching the 11 contradictions without updating the generator prompt would leave the next editorial generation pass (Mallorca refresh, Costa del Sol refresh, any new city) authored against the same prompt structure — predictable contamination at predictable rate.
- **risk_3**: Treating this as a spore #30 (memory-only) failure would lead to a prompt fix that adds WebSearch but doesn't add time-stamp clauses — catching only the subset of temporal-compositing errors that happen to be present-vs-past mismatches, missing the past-vs-past compositing class (Almudaina convent / castle).
- **risk_4**: AI-citation-pool contamination via Wikidata Q139831191 / mondmodel.com — temporally-composited claims propagate into ChatGPT/Perplexity/Gemini answers about Ibiza venues. 'Sublimotion is at Hard Rock' becoming a downstream AI answer because mondmodel.com asserted it is the worst-case scenario.

## Transfer surface

- Any editorial generation skill that produces present-tense claims about multi-year-history entities — generate-editorial, generate-destination-content, generate-intelligence-brief, generate-regulatory-brief, generate-monthly-briefs, editorial-enrichment, generate-area-editorial
- Any skill that generates 'best of <year>' or 'currently the most X' or 'this season's Y' content — these phrasings are temporal-compositing magnets
- Any new-city onboarding (CITY_ONBOARDING.md Phase 2 editorial moat) — must include time-stamp clauses in generator prompts, not just point-of-writing verification
- Any /good-spec or /prd that includes editorial deliverables — must include 'no bare time-deictics' as a Definition-of-Done criterion alongside spore #30's 'verification-at-point-of-writing'
- Any Arnold autonomous editorial task — the dispatcher should refuse editorial-class skills whose generator prompts contain bare 'now' / 'currently' constructions
- Any subagent producing memos with present-tense entity claims — CLAUDE.md §5 Form A self-invoked review must include a temporal-compositing check (any 'X is the case' claim → require date anchor or recent source)

## Distinguishing this from memory-absence failure (sibling genome)

Same outward symptom — wrong factual claims in generated prose — but a
different fix surface. Memory-absence means the fact was never available
to the generator (fix: add live web-search/retrieval to the skill).
Temporal compositing means the fact WAS available but got asserted without
a time anchor (fix: require every present-tense claim to carry an inline
"as of <date>" anchor or a source dated within 6 months of system date;
forbid bare "now"/"currently"/"this season" without a paired anchor).
Naming them separately in post-mortem keeps the conversation — and the fix
— precise; a skill can fail both classes independently.
