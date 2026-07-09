# Research Rules

The portable research-topic discipline that `rasa.module.research` installs
at `.claude/research-rules.md`. It covers the two things the module exists to
do — **track research topics** (a structured folder per ongoing
investigation) and **cross-reference / match** them (a layered linking system)
— plus the topic lifecycle, the index conventions, and the boundary against
the sibling memory modules. **Read this file when starting a topic, logging
research, linking two topics, asking "what relates to X?", or concluding an
investigation.**

This file is **Element-owned** — it refreshes on upgrade. It deliberately
does **not** decide the one thing that varies per project:

- **Where topics live, the project's tag taxonomy, and where a concluded
  research finding graduates to** (the project's authoritative layer + the
  promotion criteria).

That lives in the project-owned **`.claude/research-canon.md`** (the research
analogue of the task module's done-gate and the notes module's notes-canon).
This file references the seam; it hardcodes none of it. The **conclude /
promote** step **hard-stops** if the seam's `promotion_target` is unfilled —
guessing where a research conclusion becomes durable is the one inference this
module refuses to make.

---

## The model in one screen

Research is **ongoing, exploratory, and topic-shaped**. Unlike a task (a unit
of work that finishes) or a decision (a ruling recorded so it isn't
re-litigated), a research topic is a *living investigation* that accumulates
findings, sources, and open questions over time, and relates to other topics.

The module gives each topic a **folder with a standard structure**, keeps them
all in an **index**, and connects them with a **layered cross-reference
system**. Two capabilities, one spine:

```
   TRACK                                CROSS-REFERENCE / MATCH
   ─────                                ───────────────────────
   research/<topic-slug>/               @path        → pull a file into context
     README.md   (overview+frontmatter) [[topic]]    → the topic-to-topic graph
     log.md      (dated research log)   tags:/related: → structured matching
     findings.md (claims + evidence)
     sources.md  (references)           /xref resolves all three:
     open-questions.md                    · what links here (back-references)
   research/INDEX.md (the registry/map)   · shared-tag matches
                                          · the cross-reference map
```

---

## Capability 1 — tracking research topics

### What a topic is

A **research topic** is a folder under the project's research root (default
`research/`, overridable in the seam). One topic = one sustained question or
line of investigation. If two questions can be pursued and concluded
independently, they are two topics; if they only make sense together, they are
one topic with sections. Err toward fewer, larger topics — splitting later is
cheap, and a swarm of thin topics defeats the index.

The folder name is the **topic slug**: lowercase, hyphenated, stable, and
unique (`crystalline-substrate-spin`, `oauth-device-flow`,
`goalkeeper-sweeper-positioning`). The slug is the topic's identity — it is
what `[[wikilinks]]` and `@path` references point at, so **renaming a slug is a
breaking change** (fix the back-references, or leave a redirect stub). Pick it
carefully once.

### The per-topic structure

Every topic folder carries the same five files (scaffolded from
`.claude/research-topic-template/` by `/research new`). Not every topic fills
every file, but the shape is fixed so a reader — or a matcher — always knows
where to look:

| File | What it holds |
|---|---|
| **`README.md`** | The topic's front page: frontmatter (`id`, `status`, `tags`, `related`), the driving question, current status summary, and a distilled state-of-play. Read this first; it links to the rest. |
| **`log.md`** | The chronological research log — dated entries of what was done, read, tried, found. Append-only, newest at the bottom (or top — pick one per project and hold it). The narrative of the investigation. |
| **`findings.md`** | Durable findings: claims that survived scrutiny, each with its evidence/source and a confidence note. Distinct from the log (raw activity) — this is the distilled "what we now believe." |
| **`sources.md`** | The reading list / reference shelf: papers, URLs, files, people, datasets. Each source gets a stable local id so `findings.md` can cite it (`[S3]`). First-party / redistributable sources may be committed alongside; third-party ones are referenced, not copied. |
| **`open-questions.md`** | The live question list — what's still unknown, what to pursue next. The fuel for the next session. Questions graduate to findings (answered) or spawn new topics (too big). |

Attachments, sub-notes, and extracted data go in the topic folder as needed
(`research/<slug>/data/`, `research/<slug>/figures/`). Keep them under the
topic; the topic folder is self-contained.

### The topic lifecycle

A topic carries a `status:` in its `README.md` frontmatter. The status is the
topic's state; keep it honest:

```
open  →  active  ⇄  dormant  →  concluded  →  archived
```

| Status | Meaning |
|---|---|
| **`open`** | Registered, scoped, but not yet actively worked. A placeholder with a question. |
| **`active`** | Under current investigation — this is where logging and findings accrue. |
| **`dormant`** | Paused. Was active, set down (blocked, deprioritized, waiting on a source). Distinct from concluded — the question is still open, just not being pursued now. |
| **`concluded`** | The driving question has an answer the project accepts. Findings are distilled; ready to graduate (see "Concluding & promotion"). |
| **`archived`** | Closed and set aside — concluded-and-promoted, or abandoned. Optionally moved to `research/_archive/<slug>/`. Never deleted; the investigation survives. |

Status lives in **exactly two places that must agree**: the topic's frontmatter
and its row in `research/INDEX.md`. Nothing else records it, so they can't
drift.

### The index — `research/INDEX.md`

One index per project, the registry and map of every topic. It is
`/research`'s home view: a table of topics with status + tags + one-line
question, plus (optionally) the cross-reference map that `/xref` renders. The
index is regenerable from the topic frontmatter — treat topic frontmatter as
the source of truth and the index as the rollup. `/research new` adds a row;
status changes update the row; `/xref map` refreshes the link section.

---

## Capability 2 — cross-reference & matching (the layered link system)

The second reason this module exists: **link and match topics and research
internally.** Three mechanisms, each doing one job. Use all three; they
compose. `/link` authors them, `/xref` resolves and discovers over them.

### Layer 1 — `@path` context pointers ("pull this in")

`@`-prefixed workspace-relative paths are the **native Claude Code
file-mention convention**: when a path is written `@research/oauth-device-flow/findings.md`,
it signals "load this file into context to reason about it," and the path is
directly openable. Use `@path` when one piece of research **depends on the
actual contents** of another file — a finding that builds on another topic's
finding, a topic that must be read alongside a source document.

- Write them inline where the dependency is: in `findings.md`, `log.md`, or
  the topic `README.md`.
- Point at the **specific file** that matters (`@research/<slug>/findings.md`),
  not just the topic folder, when it's the contents you need.
- `@path` is directional and content-level. It answers "**to understand this,
  also read that.**" It is *not* the topic-graph edge — that's Layer 2.

### Layer 2 — `[[topic-slug]]` wikilinks (the topic graph)

Double-bracket wikilinks name **another topic by slug** and form the
**bidirectional topic graph** (the same convention the RasaOS memory system
uses). Use `[[slug]]` for topic-to-topic association — "this investigation is
related to that one" — independent of any single file's contents.

- `[[oauth-device-flow]]` in topic A's `README.md` asserts an edge A → B.
- The edge is **discoverable in both directions**: `/xref` reads A's outbound
  `[[...]]` links *and* finds every topic whose text contains `[[A]]` (A's
  back-references). You do not maintain the reverse edge by hand — the matcher
  computes it.
- A `[[slug]]` that names no existing topic is a **dangling link** — a marker
  that the topic is worth creating, not an error. `/xref` reports danglers so
  they become the next `/research new`.

### Layer 3 — `tags:` / `related:` frontmatter (structured matching)

The topic `README.md` frontmatter carries two structured fields the matcher
queries:

```yaml
---
id: RT-004
status: active
tags: [oauth, auth, security, protocol]
related: [oauth-device-flow, jwt-rotation]
---
```

- **`tags:`** — the project's taxonomy terms (defined in the seam). Tags drive
  **matching by shared subject**: `/xref` surfaces topics that share tags even
  when nobody linked them explicitly. This is the "matching" half of the ask —
  discovering relatedness that wasn't hand-authored.
- **`related:`** — an explicit, curated list of related topic slugs. The
  structured mirror of the `[[wikilinks]]` in the prose; `/xref` reconciles the
  two and flags disagreements (a `related:` entry with no `[[link]]`, or vice
  versa). `related:` is the machine-readable edge list; `[[...]]` is the
  in-prose one.

### When to reach for which

| You want to… | Use |
|---|---|
| Say "to understand this finding, read that file" | **`@path`** (Layer 1) |
| Assert "this topic relates to that topic" | **`[[slug]]`** in prose (Layer 2) + add to `related:` (Layer 3) |
| Discover topics on the same subject nobody linked | **`tags:`** matching via `/xref` (Layer 3) |
| Ask "what points at this topic?" | **`/xref`** back-references (computes reverse `[[...]]`) |
| See the whole graph | **`/xref map`** → renders into `research/INDEX.md` |

Rule of thumb: **`@` for content you must read, `[[ ]]`/`related` for the topic
graph, `tags` for discovery.** Author generously — links are cheap and the
matcher only helps when the edges exist.

---

## Concluding & promotion (via the seam)

When a topic reaches `concluded`, its findings often need to **graduate** out
of the research workspace into the project's durable layer — otherwise the
answer rots in a research folder nobody re-reads. **Where** a conclusion lands
is exactly the per-project thing the seam names:

- **`.claude/research-canon.md`** holds `promotion_target` (where a concluded
  finding goes), the project's `tags` taxonomy, and `promotion_criteria` (what
  makes a topic ready to conclude + graduate).

`/research conclude` reads the seam and **hard-stops if `promotion_target` is
unfilled**. It never decides on its own authority where a finding becomes
durable. Typical targets, per vertical:

- A **`rasa.module.notes` decision** (`notes/DECISIONS.md`) — the research
  produced a ruling; record it there. This is the most common target and the
  cleanest hand-off (research answers "what did we find?"; notes records "so we
  decided X").
- A **domain's `content/`** — the finding becomes reference material the domain
  ships (e.g. `domain-alignment` promoting a research result into
  `framework/`).
- A **spec or `CLAUDE.md` rule** — the finding changes how work is done.
- **Canon** (in this workspace) — where promotion additionally routes through a
  canon *task* first; the seam encodes that hard-gate.

Promotion is **staged for the human to ratify**, never auto-committed (the
substrate-wide "never auto-commit; the human commits" rule). When a finding is
promoted, note the target back in the topic (`Promoted-to:` in `README.md`) so
the chain from question → finding → durable-artifact stays traversable.

---

## The boundary against the memory siblings — read before reaching for the wrong tool

Three RasaOS primitives touch "remembering things." Conflating them is the
known failure mode:

- **`rasa.module.research` (this module)** = an **ongoing, structured
  investigation** — a folder per topic that accumulates a log, findings,
  sources, and open questions, cross-referenced into a graph. Its unit is a
  *topic under investigation*. It answers "**what are we figuring out about X,
  where does it stand, and what does it connect to?**"
- **`rasa.module.notes`** = **ordered, ratified decisions** — append-only
  institutional memory ("why did we decide X"). A *concluded* research topic
  often **promotes a finding into a notes decision** — that is the designed
  hand-off, not an overlap. Research is the workshop; notes is the ledger.
- **Kernel similarity-recall memory** (via `rasa.module.ingest` `/remember`
  + `/recall`, or the injected `memory_store` / `memory_search` tools) =
  **unordered facts recalled by vector similarity** ("what's relevant to this
  query"). A durable finding worth surfacing by meaning can *also* be
  `memory_store`'d, but this module never writes to the kernel — the
  cross-reference is soft.

Rule of thumb:

- An open question you're actively investigating, with sources and a log →
  **`/research`** (this module).
- A ruling you'll cite later to stop an argument → **`module.notes` `/decide`**.
- A self-contained fact you want auto-surfaced by similarity later →
  **`memory_store`** / `module.ingest` `/remember`.

If you find yourself building vector search or a recall-by-similarity feature
**here**, stop — that's the kernel's job (`module.ingest`). This module's
retrieval is a person (or `/xref`) reading a structured, linked corpus, not a
query against an index.

---

## Conventions (mandatory)

- **Slugs are identity.** Lowercase-hyphenated, stable, unique. Renaming
  breaks `[[links]]` and `@paths` — fix back-references or leave a redirect.
- **Status in two places, always agreeing** — topic frontmatter + `INDEX.md`
  row. Frontmatter is the source; the index is the rollup.
- **Dates absolute** (`2026-07-09`), never relative. Log entries are dated.
- **Findings carry evidence.** A claim in `findings.md` without a source or a
  stated basis is a log entry or an open question, not a finding.
- **Append-only where it's a record.** `log.md` and `findings.md` grow; you
  supersede (with a note) rather than silently rewrite history. `README.md`
  (the current-state summary) and `open-questions.md` (the live list) are
  living and may be edited freely.
- **Never delete a topic — archive it.** Set `status: archived`, optionally
  move to `research/_archive/`. The investigation, including dead ends,
  survives (dead ends are findings too: "we tried X, it doesn't work").
- **Link generously.** The matcher only helps when the edges exist. Cheap to
  add a `[[link]]` or a tag; expensive to rediscover an unrecorded connection.
- **Ratify promotions.** `/research conclude` stages; the human commits.

## Soft cross-references to sibling modules (degrade gracefully)

Present-if-mounted; never hardened into `requires.elements[]`:

- **`rasa.module.notes`** — the promotion target for concluded findings (the
  primary hand-off). `promotion_target` in the seam points here when the
  project runs notes.
- **`rasa.module.tasks`** — a topic that spawns work files a `TASK-NNN`; a task
  that needs investigation first spawns a topic. A topic `README.md` may cite
  the task, and vice-versa.
- **`rasa.module.ingest` / kernel memory** — the similarity-recall sibling (see
  the boundary section). A durable finding may also be `memory_store`'d.
- **`rasa.module.jobs`** — a scheduled "review dormant topics / stale
  open-questions" sweep is a natural `job.toml` when jobs is mounted.
