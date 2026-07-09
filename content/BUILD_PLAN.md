# `rasa.module.research` — Specification & Design Record

**Status:** v0.1.0 = a full working module (spine + seam + three skills +
per-topic templates + scaffolded index). This is the author-time design record
behind the installed spine (`content/research-rules.md`).

---

## Why this module exists

Requested directly by the workspace author (2026-07-09) to do **two
fundamental things**:

1. **Track research topics** — a new folder + standard structure per topic for
   ongoing research and development.
2. **Cross-reference / match** — link and reference topics and research
   internally, "especially using @ linking or whatever the internal Claude
   structure for this is."

Unlike `module.tasks` and `module.notes` — distillations of patterns already
reinvented across verticals, deliberately gated behind an *extract-after-proof*
rule — this module is **greenfield**. The extract-after-proof gate exists to
avoid premature abstraction of a convergent pattern; it does not apply to a
capability the author is asking to be built from scratch. So v0.1.0 ships the
working skills, not a spec-only shell.

## What "the internal Claude structure for linking" actually is (design note)

The author flagged uncertainty about the linking mechanism. The honest lay of
the land, and the decision:

- **`@path`** is Claude Code's native **file-mention** convention. In a prompt
  it expands a file into context; written in a committed file it is meaningful,
  openable text that signals "load this." It is a *content-level, directional*
  pointer — "to understand this, also read that file." **Adopted as Layer 1.**
- **`[[wikilink]]`** is the bidirectional-graph convention used by the RasaOS
  memory system (and Obsidian). It names another *topic* and forms an edge
  discoverable from both ends. **Adopted as Layer 2** (the topic graph).
- **`tags:` / `related:` frontmatter** gives a matcher structured fields to
  query — matching by shared subject (`tags`) and a curated edge list
  (`related`). **Adopted as Layer 3** (matching/discovery).

The three compose: `@` for content you must read, `[[ ]]`/`related` for the
topic graph, `tags` for discovery. The author chose the **layered** approach
(all three) over any single syntax — it's the richest and each layer does one
job. See `research-rules.md` § "Capability 2" for the authoritative model.

## Non-goals (hard boundaries)

- **Not the kernel's similarity-recall memory.** Recall-by-meaning over facts
  is `module.ingest` / `memory_store` territory. This module's retrieval is a
  person (or `/xref`) reading a **structured, linked corpus** — exact tag
  overlap + graph traversal, never vector search. If a feature here starts to
  look like similarity recall, it belongs in the kernel. (See the boundary
  section of `research-rules.md`.)
- **Not institutional memory.** A ratified decision ("why did we decide X") is
  `module.notes`. Research is the *workshop*; notes is the *ledger*. A
  concluded topic **promotes a finding into** a notes decision — the designed
  hand-off, not an overlap.
- **Not a task tracker.** Open *work* is `module.tasks`. Open *questions* live
  in a topic's `open-questions.md`; a question that becomes work spawns a
  `TASK-NNN`.

---

## Entity model

Project-owned live corpus + one project-owned seam.

### `research/<slug>/` (a topic) — five files, fixed shape

- **`README.md`** — frontmatter (`id`, `slug`, `status`, `created`, `updated`,
  `tags`, `related`) + driving question + "state of play" + scope + the
  related-topics (`[[...]]`) and context-pointer (`@path`) sections +
  `Promoted-to`.
- **`log.md`** — dated, append-only research narrative.
- **`findings.md`** — durable findings (`F#`) with confidence + evidence
  (citing `sources.md`), append-only, superseded not rewritten.
- **`sources.md`** — the reference shelf (`S#`); first-party sources may be
  committed under `sources/`, third-party referenced.
- **`open-questions.md`** — the live question list.

### `research/INDEX.md` — the registry + map
A rollup of topic frontmatter (id · slug · status · tags · question) plus the
`<!-- BEGIN/END XREF MAP -->` block `/xref map` writes into. Frontmatter is the
source of truth; the index is regenerable.

### `.claude/research-canon.md` (the seam) — the one per-project thing
`research_root`, `tags` (taxonomy), `promotion_target` (**required** for
conclude), `promotion_criteria` (optional override / hard-gate), and which
sibling modules are present.

### The lifecycle
`open → active ⇄ dormant → concluded → archived`. Status lives in exactly two
places that must agree (frontmatter + INDEX row).

---

## The three-skill contract

Thin drivers over the corpus + seam; the discipline lives in
`research-rules.md`, not duplicated per skill.

### `/research` — topic lifecycle (capability 1)
Create a topic (scaffold + register + slug discipline), log research, record
findings (evidence-gated) + sources, list/filter, change status, and
**conclude** (reads the seam, hard-stops if `promotion_target` unset, stages
the promotion for ratification).

### `/link` — author cross-references (capability 2, write side)
Add the three layers; keep `[[...]]` prose links and `related:` frontmatter in
sync; tag from the taxonomy only; ensure `@paths` resolve; re-slug fix-ups.

### `/xref` — resolve + discover (capability 2, read/match side)
The heart is what nobody stores: **back-references** (reverse `[[...]]`) and
**tag-match discovery** (related topics never explicitly linked). Plus `@path`
resolution, the graph `map` (writes only the `XREF MAP` block of INDEX), and a
`health` report (danglers / sync-mismatches / orphans). Read-only except map
and applying fixes (which stage via `/link`).

**Style/quality bar:** match `module.tasks` / `module.ingest` SKILL.md files —
a crisp operation list, honest hard-stop behavior, no invented plumbing.

---

## The seam paradox (honest note)

Pushing `promotion_target` into the seam is correct — where a conclusion
becomes durable is exactly the vertical concern. It means the module's
*horizontal* core is "the topic structure + the three-layer cross-reference
discipline," not the promotion logic. That core is genuinely load-bearing here:
the folder-per-topic shape and the compose-able link layers are the reusable
substance, and they're identical across a physics corpus, a legal matter, and a
software investigation. The vertical part is only *what you conclude into*.

---

## Version plan

- **v0.1.0 (this)** — full working module: spine + seam + three skills +
  per-topic templates + scaffolded INDEX. `bin/check-manifest` GREEN; `bin/init`
  smoke-tested. Local commit + tag; push is the author's (workspace rule).
- **v0.2.0** — hardening after a first real consumer declares the module in its
  `requires.elements[]` and runs a topic end-to-end through conclude/promote:
  refine the seam if it warps, add `/xref` output polish, optional
  `research/_archive/` convention if the move-on-archive proves needed.
- **v1.0.0** — the seam format + topic structure + link model locked after at
  least two real verticals run the pipeline unchanged (the decisive test: does
  the spine survive a second, divergent consumer without reshaping?).

## First consumers (natural)

- **this workspace** (`rasa.tenant.rasaos`) — the orchestrator already runs
  ad-hoc investigations; dogfooding would prove the module against canon work.
- **`domain-alignment`** — a research-heavy corpus domain (CKR physics) with a
  real, growing body of interlinked topics and sources; the richest test of the
  cross-reference/matching layer.
