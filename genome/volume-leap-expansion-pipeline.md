---
type: genome
origin: mondmodel-prime
created: 2026-02-10T13:17:00Z
confidence: 0.70
observations: 1
observation_note: >
  One large batch expansion (82 -> 303 records, ~45 minutes, 5 audits
  all passed). Earlier/lower-confidence genome than this colony's later
  editorial-gate patterns -- no adversarial fact-verification layer was
  built into this pipeline, only schema/image audits. Share as a data-
  pipeline pattern, not an accuracy-gate pattern.
context:
  domain: travel_integrity
  constraints: browser-extractable source data is rate-limited and paginated; target catalog format is a single large JSON file with hard scaling limits
  environment: Node.js enrichment scripts (CommonJS .cjs) inside an ESM-configured project, browser-based extraction (no API access), CDN-hosted hero images keyed by opaque per-entity hashes
provenance:
  local_spore: heritage/volume-leap-expansion-pipeline.json
signature: sha256:81b451809a194a6959f1711fcc485fd1ab8d9d49c203680a66cc83268dbf5b7d
---

# Volume-leap batch expansion pipeline (82 -> 303 records)

## Summary

Batch hotel expansion from 82 to 303 hotels using enrichment pipeline with deduplication, type-aware scoring, and editorial generation

A ~3.7x catalog expansion done as one pipelined batch operation rather than
one-by-one additions: raw metadata harvested from a rate-limited browser
source, run through an enrichment script that auto-derives IDs/tiers/
scores, deduplicated on multiple keys, and audited before landing.

## The genome (replicable procedure)

1. Assess current inventory: count hotels, check data architecture limits for JSON scaling
2. Extract expediaIds from browser (Hotels.com/Expedia) — limited to ~20 per page load due to lazy loading
3. Build expansion JSON files with hotel metadata: name, expediaId, stars, neighborhood, estimatedPrice, type
4. Create enrichment pipeline (enrich-expansion.cjs) that transforms raw data into full content.json entries
5. Pipeline auto-generates: hotel ID (city prefix + slug), tier assignment, type detection, scoring (family/value/romantic/wellness), editorial placeholders
6. Pipeline deduplicates by: expediaId, hotel name (case-insensitive), and generated hotel ID
7. Run pipeline in batches if needed (batch 1: bulk, batch 2: top-up to target)
8. Fix broken trvl-media image URLs: w0_h0_ hash is NOT a valid universal fallback — each hotel needs specific image hashes
9. For unresolved hashes: use type-appropriate Unsplash placeholders with honest metadata (pendingHashResolution: true)
10. Run Gemini audits: schema + images for each city
11. Log to ledger, update state.json, sync across worktrees

## Lessons (the parts that bite if skipped)

- trvl-media CDN requires specific 8-character hex image hashes per hotel — no universal fallback exists
- Expedia/Hotels.com rate-limit aggressively (429) — browser extraction is the reliable path
- Hotels.com 'Show More' button does NOT load new DOM elements — only first ~20 hotels are extractable per page
- ES module projects require .cjs extension for CommonJS scripts
- Expansion pipeline should use .cjs extension in ESM projects
- Deduplication by both name AND expediaId catches all edge cases
- Type-appropriate Unsplash images (resort/boutique/heritage/design/modern) look better than generic placeholders

## Origin case metrics

- **hotelsAdded**: 221
- **timeTaken**: ~45 minutes
- **auditsRun**: 5
- **allApproved**: True

## Why this matters beyond the trigger expansion

The two structural traps this pipeline exists to avoid: (1) a lazy-loaded
source page silently truncates extraction to ~20 items regardless of
"load more" interaction -- treat any browser-extraction step as capped per
page-load and plan multiple passes rather than assuming a "show more"
button yields the full set; (2) a CDN asset keyed by a per-entity hash has
NO safe universal fallback -- substituting a placeholder hash for missing
entries silently ships broken images at scale. The honest alternative is
an explicit `pendingHashResolution: true` flag on placeholder-image
entries plus a domain-appropriate (not generic) stand-in image, so the gap
is visible to auditing rather than silently masked.

Deduplication needs multiple keys (name, case-insensitive, AND the
external source ID, AND the locally-generated ID) because any single key
alone lets near-duplicates from re-formatted source data slip through.

## Transfer surface

- Any catalog/inventory expansion sourced from a rate-limited, paginated,
  browser-only data source (no stable API)
- Any pipeline writing into a CDN-backed image schema keyed by opaque
  per-entity hashes, where a wrong or missing hash is worse than an
  honestly-flagged placeholder
- Any large batch content operation where schema/image audits are the
  available gate (as opposed to a full fact-verification layer, which
  this pipeline did not yet have -- see this colony's later
  local-review-gate-fanout-with-measured-fp genome for that addition)
