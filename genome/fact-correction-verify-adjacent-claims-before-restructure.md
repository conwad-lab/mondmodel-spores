---
type: genome
origin: mondmodel-prime
created: 2026-04-23T00:30:00Z
confidence: 0.80
observations: 1
observation_note: >
  Single well-instrumented incident (one artifact, one factual trigger,
  full verification trail against a primary source). Not yet replicated
  across multiple independent artifacts, hence confidence below the
  higher-observation genomes already shared by this colony.
context:
  domain: editorial
  constraints: structural fix driven by a single flagged factual claim, in an artifact containing OTHER factual claims at the same specificity level
  environment: multi-locale editorial content with a numeric/named-entity claim appearing in both a summary position (lede, title, bullet count) and an elaborated position (body list)
provenance:
  local_spore: heritage/fact-correction-verify-adjacent-claims-before-restructure.json
signature: sha256:3eb682a01bc1379a83926a05569fe7eadeb8f5bada65c19571d2b455c78031f2
---

# Verify adjacent claims before restructuring around one correction

## Summary

Before applying a structural fix driven by a single factual correction, re-verify ALL adjacent factual claims. A coincidentally-correct claim elsewhere in the artifact may constrain (or enable) the minimal-change fix path. A rejected substitution candidate may itself be factually wrong and must be verified before being ruled out.

Trigger incident: a dining guide listed a restaurant as 2-star Michelin
in a "Two-Star Tier" section; verification against the primary source
showed it was actually 1-star (held continuously for over a decade). The
naive fix -- move it to the correct tier -- is right, but choosing HOW to
restructure required checking every OTHER star claim in the same artifact
first, including a candidate substitute the fix-prompt suggested, which
turned out to ALSO be misclassified.

## The genome (replicable procedure)

1. Receive structural-fix chip driven by a single factual claim (e.g., 'X is likely wrong — verify before rewriting prose')
2. Verify the load-bearing claim against the primary source (in this case: Michelin Guide España 2026 via guide.michelin.com + ara.cat + visitacostabrava.com)
3. BEFORE choosing a fix option: re-verify EVERY adjacent factual claim in the same artifact at the same level of specificity. In this case: all 11 other named Michelin-starred restaurants in the one-star constellation + the 3 remaining two-stars + the lede's numeric counts.
4. Re-verify the REJECTED substitution candidates from the chip prompt before ruling them out. The chip suggested Les Magnòlies (Arbúcies) as a possible 2-star substitute — verification showed Les Magnòlies is 1-star in the 2026 guide. Had that verification been skipped, option (b) could have been applied and would have introduced a second factual error.
5. Cross-check the TRUTH-map (e.g., costa-brava-context.json gastronomy.michelin_2025_2026). A missing entry in the TRUTH-map is itself a factual-integrity signal.
6. Examine the lede and summary claims. If the lede's numeric count ('tre tvåstjärniga, tolv enstjärniga' / 'three two-stars, twelve one-stars') is coincidentally already correct — option (c) 'demote and propagate' becomes the minimal-change fix. The lede's coincidental correctness is load-bearing for the decision tree.
7. Apply the structural fix: move/rename/demote in the affected section, propagate the change to any section that names the moved entity, update the TRUTH-map.
8. Only AFTER structural verification is complete: proceed with prose rehabilitation (diacritic restoration, naturalness, voice contract), because rehabilitating corrupted prose around a still-wrong structural claim would lock in the error.
9. Route through editorial-steward for APPROVE_WITH_REVISIONS gate on the rehabilitated prose.
10. Write a dedicated post-steward polish ledger entry (per ledger-0145 precedent) for the audit trail.

## Decision tree observed

- Option A (rejected): rename the section and restructure around actual counts -- more invasive than needed.
- Option B (rejected via verification): substitute a different entity into the vacated slot -- the suggested candidate was itself factually wrong; verifying it before ruling it out prevented trading one error for another.
- Option C (chosen): demote the misclassified entity to its correct tier and propagate the change everywhere it's named -- chosen because a coincidentally-already-correct summary count (lede) made this the true minimal-change path, which was only knowable after verifying the summary claim too.

## Gates that confirmed the fix was complete

- Michelin Guide España 2026 direct verification (12 of 12 one-stars cross-checked)
- TRUTH-map alignment (costa-brava-context.json updated with missing Casamar entry + Fina Puigdevà → Fina Puigdevall typo fix)
- editorial-steward APPROVE_WITH_REVISIONS (3 actionable SV findings applied)
- Ledger chain verified (39 clean entries from anchor ledger-0118)
- npm run build:local clean (1m 1s)

## Counterfactuals (what happens without this genome)

- **risk_1**: Skipping step 3 (adjacent verification): could have rewritten only Section 2 to three two-stars without realizing Section 3's 11-name list was missing Casamar — leaving the lede's 'tolv enstjärniga' claim still unsupported.
- **risk_2**: Skipping step 4 (verifying rejected candidates): applying option (b) would have substituted Les Magnòlies as a 2-star when it is in fact 1-star — trading one error for another.
- **risk_3**: Skipping step 6 (lede coherence check): could have chosen option (a) 'rename section' as the minimal-risk path, when in fact option (c) was the minimal-change path because the lede was already correct.
- **risk_4**: Skipping step 8 (structure before prose): rehabilitating SV diacritics on a still-corrupt Section 2 would have produced beautifully-rendered but factually-wrong Swedish prose — harder to catch in review because the visible flaws would be resolved.

## Transfer surface

- Any article or editorial artifact where a single factual error is flagged
- Any TRUTH-map update driven by a downstream content correction
- Any multi-locale rehabilitation where EN is treated as ground truth (verify EN is actually correct before promoting it)
- Any numeric claim (count, price, year, distance, star rating) that appears in both a summary position (lede, title, bullet) and an elaborated position (body)

## Why this matters beyond the trigger incident

A factual-correction chip usually arrives scoped to ONE claim. The natural
failure mode is fixing exactly that claim and stopping -- but (a) claims at
the same specificity level nearby are often wrong for the same underlying
reason (stale source, transcription drift) and go uncaught, (b) any
substitute or alternative the fix-prompt proposes is itself an unverified
claim, not a known-good replacement, and (c) summary-position claims (lede
counts, title numbers) are the cheapest tell for which structural fix is
actually minimal -- check them before picking a fix path, not after.
