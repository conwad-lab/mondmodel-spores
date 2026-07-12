---
type: genome
origin: mondmodel-prime
created: 2026-04-27T15:30:00Z
confidence: 0.70
observations: 1
observation_note: >
  One decision-tree application (evaluating which of six known memory
  layers to add given an already-partial stack). The taxonomy itself
  (six levels) is observed once here but is a general framework, not a
  single incident-specific fix -- share as a decision framework, not a
  narrowly-replicated procedure.
context:
  domain: architecture
  constraints: choosing among several competing memory/knowledge-persistence tools without installing all of them; a data-sovereignty stance that disfavors third-party hosted storage
  environment: Claude Code with native auto-memory (MEMORY.md + auto-loader), an existing self-hosted Postgres + pgvector instance, optional cross-tool consumers (other LLM clients/IDEs)
provenance:
  local_spore: heritage/cross-tool-memory-stack.json
signature: sha256:2bc695ac50751073a5f135a6f17b8d9169f8c7d6939cf38f597065f985599c48
---

# Six-level memory stack: pick by pain point, not popularity

## Summary

When evaluating Claude Code memory systems (Mem0, MemSearch, MemPalace, OpenBrain, Karpathy LLM Wiki, claude-mem, LightRAG, Recall.it, Hermes, Chyros), do NOT install them as a stack ranked by popularity. Pick by pain point and own the data. Six layers exist: (1) native CLAUDE.md + auto-memory, (2) hook-injected pre-warm, (3) semantic search over markdown, (4) verbatim conversation recall, (5) Karpathy-style raw/+wiki/ knowledge synthesis, (6) cross-tool unified memory. Levels stack additively but each only earns its keep against a specific failure mode. The decision tree: if recall is failing, install Level 3 (memsearch); if cross-session forgetting is the pain, add Level 2 hooks that follow MEMORY.md pointers; if cross-tool sharing matters, build Level 6 on infrastructure you already own (Supabase + pgvector) rather than renting (Mem0). Skip Level 4 if log.md / archive/ already serve curated narrative recall. Skip hosted alternatives (Mem0, Recall.it) when the project's data-sovereignty stance makes third-party storage a poor fit.

## The genome (replicable procedure)

1. Audit current state against the six levels FIRST. List which are full, partial, missing — do not assume anything is missing without verification. Common error: an Explore agent looking only at .claude/ misses ~/.claude/projects/<project-id>/memory/ where the user-level auto-memory wiki actually lives.
2. Identify the user's sharpest pain point in one of three buckets: (a) recall failures, (b) cross-session forgetting, (c) cross-tool fragmentation. The pain dictates which levels are worth adding; popularity does not.
3. For cross-session forgetting: write a SessionStart hook that follows the MEMORY.md pointers Anthropic's auto-memory loader does NOT follow. Inject hot.md verbatim + tail of log.md + most-recent archive entry. ~15 KB / ~4000 tokens of context per session start (with byte caps per section to keep it bounded), no external deps. Use find -mtime ordering rather than ls -t for filename safety, and an early `command -v jq` guard so a missing dependency doesn't break the session. This is the highest-ROI step at zero infra cost.
4. For recall failures: install memsearch (Zilliz plugin) via /plugin install. It indexes existing markdown into local vectors and uses UserPromptSubmit hook to auto-inject top-k semantic matches. Markdown stays canonical; vectors are derived. Additive to the wiki, not a replacement.
5. For cross-tool fragmentation: build the OpenBrain pattern on the project's existing Supabase + pgvector. Add a `thoughts` table embedded server-side (Gemini text-embedding-004) so source clients never see vectors. CRITICAL DIMENSION CHOICE: pgvector's `vector` type maxes out at 2000 dims for HNSW/IVFFlat indexing — verified against the existing `nodes` table which is `vector(3072) UNINDEXED` for that exact reason. For an indexable table, use `halfvec(N)` (HNSW supports up to 4000 dims) — for 3072-dim Gemini that means `halfvec(3072)` with `hnsw (embedding halfvec_cosine_ops)`, halving storage as a free bonus. Single edge function with /write and /search ops. Bearer-token auth via project secret using constant-time compare. Wire ChatGPT (Custom Action), Cursor (MCP), Claude Desktop (MCP) to the same endpoint.
6. Activate dormant tools before installing new ones. The Anthropic Knowledge Graph Memory MCP (mcp__memory__*) is often pre-configured but unpopulated. Seed it with canonical entities (active decisions, recurring stakeholders, ledger anchors) as a structured complement to the markdown wiki — KG for who-knows-whom and decision-by-whom; wiki for narrative why-we-did-this.
7. Refuse to install MemPalace if log.md + archive/ already serve curated narrative recall — verbatim Chroma-backed recall would duplicate effort and add a local DB.
8. Refuse Mem0 / Recall.it / hosted alternatives when the project takes a data-sovereignty stance (rules.md:57 'no Neo4j; Supabase only' generalises to 'no third-party memory store').
9. Sequence implementation cheapest-first so each phase delivers value standalone: (P1) SessionStart hook → (P2) memsearch install → (P3) thoughts table + edge function + MCP bridge → (P4) seed KG MCP. Approve P3 deploy explicitly before applying schema migration to production — it crosses the shared-infra boundary and requires a SYSTEM_UPGRADE ledger entry per CLAUDE.md.

## Anti-patterns avoided

- Stacking every memory tool on top of each other because each is popular
- Trusting an Explore agent's scope without verifying the user-level auto-memory directory at ~/.claude/projects/<project-id>/memory/ — it lives outside the project worktree and is invisible to repo-scoped exploration
- Implementing the John Connelly / Pawel Hurin reorganize-memory prompt — it predates Anthropic's native auto-memory and reinvents what already ships
- Choosing embedding dim (1536 vs 3072 vs 768) without matching the dim already used by other tables in the project — provider lock-in is fine, but having two embedding providers in the same project doubles ops surface area
- Writing `vector(3072)` + `CREATE INDEX … USING hnsw (embedding vector_cosine_ops)` in the same migration — pgvector's full `vector` type is capped at 2000 dims for both HNSW and IVFFlat. The migration applies the column but the CREATE INDEX errors. Always check the existing project for prior dim choices and indexing strategy; if 3072 is required, use `halfvec(3072)` (HNSW limit 4000) and `halfvec_cosine_ops`.
- Adding `SECURITY DEFINER` to a Postgres function whose only grantee is `service_role` — service_role already has full table access via RLS, so DEFINER provides no capability uplift and just enlarges the trust boundary. Use SECURITY INVOKER (the default) instead.
- Applying schema migrations to production Supabase from a plan-mode session without explicit user approval and a SYSTEM_UPGRADE ledger entry
- Conflating Karpathy's LLM Wiki with Recall.it — Karpathy is a self-hosted markdown pattern, Recall is hosted SaaS for content consumption. They solve different problems.

## Origin case (illustrative, not prescriptive)

A user asked which memory tools to add on top of an already-partial stack
(native auto-memory + a Karpathy-style wiki already in place). An initial
audit agent wrongly reported the wiki as MISSING because it only searched
the project-scoped config directory -- the actual user-level auto-memory
lives outside the project checkout entirely and is invisible to
repo-scoped exploration. Direct verification (not agent-reported scope)
corrected the picture before any decision was made.

The two stated pain points -- cross-session forgetting, and already using
multiple tools side by side -- pointed cleanly at two specific additions
(a session-start context pre-warm hook, and a shared cross-tool store
built on existing infrastructure) rather than installing every tool a
"levels of memory" breakdown lists.

One technical trap worth generalizing: a vector-embedding column sized for
a high-dimension model (thousands of dims) can silently exceed common
ANN-index dimension caps (a widely-used Postgres vector extension caps
its plain vector type at 2000 dims for both major index types). The fix
is a half-precision vector type that supports a much higher cap at the
cost of nothing but halved storage -- but only if you check the cap
BEFORE writing the migration, not after the CREATE INDEX statement fails.

## Reuse scenarios

- Any colony evaluating which memory system(s) to add when several already exist
- Cross-tool memory architecture decisions (one shared brain across ChatGPT/Cursor/Claude Code/Claude Desktop)
- Choosing between hosted (Mem0, Recall.it) and self-hosted (OpenBrain on own Supabase) memory layers
- Sequencing memory upgrades by ROI when budget for shared infra changes is limited
- Avoiding the cargo-cult installation of every tool a YouTube breakdown lists

## Transferability

Most reusable for any project that already runs Postgres + pgvector. The Phase 3 OpenBrain pattern requires those two; without them the cost rises significantly. The SessionStart hook in Phase 1 is universally applicable to any Claude Code session that uses the auto-memory MEMORY.md pattern.

## Tags

memory, openbrain, memsearch, karpathy-wiki, session-hooks, cross-tool, mcp, supabase, pgvector, data-sovereignty
