# `rasa.module.research` — content

What this module ships and where it installs. This file is author-time
documentation (not installed into consumer projects).

## The one-liner

The **research** module: a portable, folder-per-topic workspace for ongoing
investigation, plus a layered cross-reference/matching system that links and
discovers relationships between topics.

## What installs where

| Source | Installs to | Policy | What it is |
|---|---|---|---|
| `content/research-rules.md` | `.claude/research-rules.md` | file-replace | The spine — topic model + structure, lifecycle, the three-layer cross-reference/matching model, INDEX conventions, promotion flow, memory-sibling boundary. Element-owned; refreshed on upgrade. |
| `content/topic-template/` | `.claude/research-topic-template/` | directory-mirror | The five per-topic skeleton files `/research new` instantiates into `research/<slug>/`. Element-owned. |
| `content/skills/` | `.claude/skills/` | directory-mirror | `/research`, `/link`, `/xref`. Element-owned. |
| `seed/research-canon.md.template` | `.claude/research-canon.md` | skip-if-exists | **The seam** — research root + tag taxonomy + promotion target. Project-owned; `/research conclude` hard-stops until `promotion_target` is filled. |
| `seed/research/INDEX.md.template` | `research/INDEX.md` | skip-if-exists | The topic registry + graph map. Project-owned; also creates the `research/` root. |
| `seed/rasa.lock.json.template` | `.claude/rasa.lock.json` | init-only-with-sha | Connection-Contract lockfile, SHA-stamped at init. |

## The two capabilities (the reason it exists)

1. **Track research topics** — each topic is a `research/<slug>/` folder with a
   fixed five-file structure (`README` overview + `log` + `findings` +
   `sources` + `open-questions`), an `open→active⇄dormant→concluded→archived`
   lifecycle, and a row in the regenerable `research/INDEX.md`.
2. **Cross-reference / match** — three layers that compose:
   - `@path` — pull a specific file into context (Claude Code's native
     file-mention convention).
   - `[[topic-slug]]` — the bidirectional topic graph.
   - `tags:` / `related:` frontmatter — structured matching.
   `/xref` computes **back-references** (the reverse of `[[...]]`, which nobody
   stores) and **tag-match discovery** (related topics nobody explicitly
   linked), plus a graph map and a health report.

## The three skills

| Skill | Side | Does |
|---|---|---|
| `/research` | track | create a topic, log research, record findings/sources, change status, conclude (stage promotion) |
| `/link` | write cross-refs | add the three link layers, keep `[[...]]`↔`related:` in sync, tag topics |
| `/xref` | read/match cross-refs | back-references, tag-match discovery, render the map, health report |

## The shape

Toolkit module, `requires.parent_kind: [domain, orchestrator]`. Pure
Element-layer convention — no kernel engine. Cross-refs to `module.notes` (the
promotion target for a concluded finding), `module.tasks`, `module.ingest`
(kernel similarity-recall — a *different* primitive), and `module.jobs` are all
soft (present-if-mounted; never in `requires.elements[]`).

## See also

- [`BUILD_PLAN.md`](BUILD_PLAN.md) — the full spec + design record.
- `content/research-rules.md` — the installed spine (the discipline).
- `../module-notes/` — the institutional-memory sibling; a concluded research
  finding **promotes into** a notes decision. The boundary is load-bearing.
- `../module-ingest/` — wraps the kernel's *similarity-recall* memory; a
  different primitive from this module's linked-corpus retrieval.
- Canon `ELEMENT_CONTRACT.md` §7 — install policies.
