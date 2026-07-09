# Rasa · Module · Research

**Canonical name:** `rasa.module.research`
**Repo / folder:** `module-research`
**Kind:** `module` (canon Spec §6)
**Contract:** Element Contract v1.3.0
**Version:** 0.1.0 (full working module — spine + seam + three skills)
**Status:** Built locally; not yet pushed. Target: `RasaOS/module-research`.

## What this is

A portable **research workspace**, mountable into any domain or orchestrator.
It does two fundamental things:

1. **Tracks research topics** — every topic is a `research/<slug>/` folder with
   a standard five-file structure and a lifecycle, indexed in
   `research/INDEX.md`.
2. **Cross-references & matches** — a layered linking system connects topics and
   surfaces relationships, using `@path` pointers, `[[topic]]` wikilinks, and
   `tags`/`related` frontmatter.

## The two capabilities

### 1 — Track

Each topic is a folder with a fixed shape:

```
research/
  INDEX.md                    # registry + graph map (a rollup of frontmatter)
  <topic-slug>/
    README.md                 # frontmatter + driving question + state of play
    log.md                    # dated research narrative (append-only)
    findings.md               # durable findings + evidence (append-only)
    sources.md                # the reference shelf
    open-questions.md         # the live question list
```

Lifecycle: `open → active ⇄ dormant → concluded → archived`. Status lives in
the topic frontmatter and its `INDEX.md` row (which must agree).

### 2 — Cross-reference / match (three layers)

```
@research/<slug>/findings.md   → pull a file into context   (Layer 1: content)
[[topic-slug]]                 → the bidirectional graph     (Layer 2: topics)
tags: [...] / related: [...]   → structured matching         (Layer 3: discovery)
```

`/xref` computes **back-references** (the reverse of `[[...]]`, stored nowhere)
and **tag-match discovery** (related topics nobody explicitly linked), renders
the graph map, and reports broken/dangling links.

## The three skills

| Skill | Does |
|---|---|
| `/research` | create a topic, log research, record findings/sources, change status, conclude (stage promotion) |
| `/link` | author the three cross-reference layers; keep `[[...]]`↔`related:` in sync; tag topics |
| `/xref` | back-references, tag-match discovery, the graph map, health report |

## The seam (the one per-project thing)

Where topics live, the tag taxonomy, and **where a concluded finding graduates**
are project-owned, in `.claude/research-canon.md` (seeded skip-if-exists).
`/research conclude` hard-stops until `promotion_target` is filled — guessing
where a finding becomes durable is the one inference the module refuses to make.

## Boundary (read this)

- **Not institutional memory.** A ratified decision ("why did we decide X") is
  `rasa.module.notes`. A concluded research topic **promotes a finding into** a
  notes decision — the designed hand-off. Research is the workshop; notes is the
  ledger.
- **Not the kernel's similarity-recall memory.** Recall-by-meaning over facts is
  `rasa.module.ingest` / `memory_store`. This module's retrieval is a person (or
  `/xref`) reading a structured, linked corpus — exact tag overlap + graph
  traversal, never vector search.

## Install

`bin/init [target-dir]` reads `rasa.json` and applies each `element.files[]` /
`seed.files[]` entry by its declared policy (no hardcoded file list). Installs
the spine + topic templates + skills into `.claude/`, seeds the seam + index +
lockfile.

## See also

- `content/research-rules.md` — the installed spine (the discipline).
- `content/BUILD_PLAN.md` — the full spec + design record.
- `../module-notes/` — the institutional-memory sibling (the promotion target).
- `../module-ingest/` — the kernel-similarity-recall sibling (a *different*
  primitive).
- `../REGISTRY.md` — the live workspace snapshot.
