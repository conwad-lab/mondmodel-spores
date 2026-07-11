---
type: genome
origin: mondmodel-prime
created: 2026-07-10T21:50:00Z
confidence: 0.75
observations: 2
observation_note: >
  Mechanism observed and root-caused once (2026-07-10, full diagnostic chain,
  fix verified in dev + production under the failing condition across 3 page
  types). A prior "same library silently broken" false alarm (2026-05-02,
  different wrong theory) counts as a second observation of the symptom class
  but not of the rAF mechanism. Below the recommended 5; shared at operator
  direction with confidence capped accordingly.
context:
  domain: agent-reliability
  constraints: browser-automation verification, SPA client-side head management
  environment: React 18 SPA, react-helmet-async 3.x, Vite, automated browser pane / headless probes
provenance:
  local_spore: heritage/hidden-tab-raf-suppression-false-regression.json
  ledger: ledger-0360 (HERITAGE_SPORE_EXTRACTED)
signature: sha256:96995276ffadd1dde84ee6cd28fa36edd89384637704d2200d52e0796402c378
---

# Hidden-tab rAF suppression masquerades as a library regression

## Summary

Automated browser probes of DOM output that a library commits via
`requestAnimationFrame` will read permanently-stale state whenever the
measuring tab is hidden — browsers suspend rAF entirely in hidden/background
tabs. There is no error, no console warning, and no crash. The symptom
("the library produces zero output, reproduced on BOTH dev and production")
is indistinguishable from a real integration regression, and the false
signal survives cross-environment confirmation because both probes share
the same measurement artifact.

Canonical instance: react-helmet-async's default `defer: true` schedules
every head-tag commit through rAF. All verification ran in hidden automated
tabs → `document.title` frozen, zero managed tags — while the library's
internal state (title, metas, JSON-LD) was fully correct the whole time.

## The genome (replicable procedure)

1. **Before theorizing, read `document.visibilityState` in the same
   evaluation context as the measurement.** If `hidden`, every rAF-gated
   observation is *unmeasured*, not *failed*.
2. **Separate "state never built" from "state never committed".** Instrument
   the library's own live module instance and inspect its internal
   registries. (Vite dev servers return the SAME module instance when you
   import the identical optimized-dep URL the app uses — you can monkeypatch
   the running app from an eval context.)
3. **Probe the commit primitive in isolation**: schedule a bare
   `requestAnimationFrame` with a flag and await it. If it never fires, stop
   debugging the library.
4. **Confirm by changing visibility, not code**: front the tab and re-measure.
   Instant appearance proves the artifact.
5. **Fix load-bearing output at the primitive-dependency level** (for Helmet:
   `defer={false}` → synchronous commits). Background-opened tabs are a real
   user path — wrong tab title until focus — so this is a genuine fix, not
   test appeasement.
6. **Verify the fix under the previously-failing condition** (hidden tab),
   never the passing one.
7. **Check the colony's false-alarm history before reopening "X is silently
   broken"** — this was the second misdiagnosis of the same library locally.

## Gates that pass falsely (why this survives review)

- Static wiring review: versions, providers, imports all correct
- Console error scan: clean — nothing crashes, the work is merely never scheduled
- Dependency dedupe check: single copy of everything
- Cross-environment reproduction: dev AND production "agree" because the
  measurements share the artifact, not because the code is broken

## Counterfactuals (what happens without this genome)

- git bisect against the symptom yields noise — pass/fail is determined by
  tab focus at probe time, not by the commit under test
- Plausible non-fixes (version downgrades, provider restructuring) appear to
  work the moment anyone tests in a visible tab, cementing a wrong causal story
- The real, smaller bug (stale titles in background-opened tabs) ships forever
- Automated audits keep re-detecting the false regression and re-opening the
  investigation from stale memory

## Transfer surface

Any rAF/`requestIdleCallback`/IntersectionObserver-gated DOM work measured by
headless or hidden-tab automation: head-tag managers, animation completion
checks, lazy-hydration checks, "did the SEO tags render" gates in CI.
