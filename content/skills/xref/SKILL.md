---
name: xref
description: Cross-reference resolver and matcher for research topics. Answers "what links to this topic?" (back-references — the reverse of [[wikilinks]]), surfaces related topics that share tags but were never explicitly linked (matching/discovery), renders the whole topic graph as a map into research/INDEX.md, resolves @path pointers, and reports dangling links and [[...]]-vs-related: mismatches. Read-only over the corpus except when writing the map or a health report. Triggered when the user wants to traverse or audit the research graph — e.g. "/xref", "what links to the oauth topic", "what else is tagged security", "show the research map", "find broken links", "what's related to this that I haven't connected".
---

# /xref — Cross-reference resolver & matcher

You resolve and discover connections across the research corpus for
`rasa.module.research` — the **read/match** side of cross-referencing.
Companion to `/link` (which authors the edges) and `/research` (which tracks
the topics). Where `/link` writes edges, you *traverse* them, *compute the
reverse* of them, and *discover* relationships nobody authored.

The link model — the three layers, back-references, and tag-matching — is
defined in **`.claude/research-rules.md`** (§ "Capability 2"). This skill
computes over that model. The research root and tag taxonomy come from
**`.claude/research-canon.md`**.

Most operations are **read-only**. The two that write —
`map` (refresh the graph section of `INDEX.md`) and `health` fixes — stage
edits for the human, never auto-commit.

## What you compute (the model)

- **Forward links** — a topic's own outbound `[[slug]]`, `related:`, and
  `@path` references (just read them).
- **Back-references** — the *reverse* edge, which nobody stores: every topic
  whose text contains `[[X]]` or `@research/X/...` is a back-reference of X.
  You compute this by scanning the corpus. This is the heart of the skill —
  "what points at this?" is answered nowhere else.
- **Tag matches (discovery)** — topics that share `tags:` but have no explicit
  `[[link]]`/`related:` edge. These are *candidate* relationships the author
  never drew — surfacing them is the "matching" half of the module's purpose.
- **Graph health** — dangling links (a `[[slug]]`/`@path` with no target),
  `[[...]]`-vs-`related:` mismatches, and orphan topics (no links in or out).

## Behavior contract

- **Scan the whole research root.** Back-references and tag-matches require
  reading every topic's `README.md` frontmatter + the files that carry links
  (`README.md`, `findings.md`, `log.md`). Read `INDEX.md` for the roster;
  read the topic frontmatter as the source of truth (the index is a rollup).
- **Report both directions.** For any topic, forward links AND back-references
  — the reverse edge is the whole point; never report only what a topic
  declares about itself.
- **Distinguish authored from discovered.** Explicit `[[link]]`/`related:`
  edges are *authored*; shared-tag matches are *discovered candidates*. Label
  them differently — never present a tag-match as if the author drew it.
- **Read-only by default.** Only `map` and applying a `health` fix write, and
  both stage for the human.
- **Cite locations.** Every link you report names the file it lives in
  (`research/<slug>/findings.md`) so the user can jump to it.

## Common operations

### Operation 1 — Back-references ("what links to X?")

1. **Confirm the target topic** X (by slug).
2. **Scan the corpus** for `[[X]]` (prose graph edges) and `@research/X/...`
   (context pointers) in every topic's `README.md`, `findings.md`, `log.md`,
   and in `INDEX.md`.
3. **Report the inbound set**, grouped by kind (topic-graph back-refs vs
   context-pointer back-refs), each with its source location and the one-line
   context around it.
4. **Also show X's forward links** for the full two-directional picture.
   Read-only.

### Operation 2 — Tag-match discovery ("what's related that I haven't linked?")

The matching capability: find relatedness nobody authored.

1. **Read the target topic's `tags:`** (or take a tag/tag-set from the user).
2. **Scan all topics' frontmatter** for shared tags. Rank candidates by number
   of shared tags (most overlap first).
3. **Subtract already-authored edges** — a candidate that's already in the
   target's `related:`/`[[links]]` isn't a discovery; mark it "already linked."
   The interesting output is *high tag-overlap, no explicit link*.
4. **Report the candidates** with their shared tags and a one-line "why they
   might connect," and offer: "author any of these edges? → `/link`."
   Read-only.

### Operation 3 — Resolve `@path` pointers

1. **Collect the `@path` pointers** in a topic (or the whole corpus).
2. **Check each resolves** (workspace-relative target exists). Report broken
   ones as danglers (Operation 5).
3. **Optionally render** the pointed-at content inline so the user sees what a
   topic pulls in. Read-only.

### Operation 4 — Render the research map (writes `INDEX.md`)

1. **Build the graph** from every topic's outbound `[[links]]`/`related:` +
   computed back-references.
2. **Render** an adjacency view — per topic: status, tags, → outbound links,
   ← back-references — plus a "clusters" grouping by dense tag/link overlap.
3. **Write it** into the `<!-- BEGIN XREF MAP -->` / `<!-- END XREF MAP -->`
   block in `research/INDEX.md` (create the block if absent). Only that block
   is touched. Stage for the human; don't finalize.

### Operation 5 — Graph health report

1. **Danglers** — every `[[slug]]`/`@research/slug/...`/`@path` whose target
   doesn't exist. A dangling `[[slug]]` is a "create this topic next" signal,
   not an error — present it that way; a dangling `@path` to a moved/renamed
   file *is* a bug to fix.
2. **Sync mismatches** — a `[[slug]]` in prose with no matching `related:`
   entry (or the reverse). List each; offer to reconcile via `/link`.
3. **Orphans** — topics with no inbound and no outbound links. Surface them:
   either they need connecting (`/link`) or they're genuinely standalone.
4. **Report** grouped by severity. Applying a fix stages an edit (via `/link`);
   don't finalize.

## What you must NOT do

- **Don't present a tag-match as an authored link.** Discovered candidates and
  authored edges are different things — label them so.
- **Don't report only forward links.** Back-references are the reason this
  skill exists; always compute the reverse edge.
- **Don't auto-fix silently.** Danglers and mismatches are *reported*; fixes
  go through `/link` and the human commits. A dangling `[[slug]]` may be an
  intentional "to create" marker — never delete it to "fix" the report.
- **Don't write outside the `XREF MAP` block** of `INDEX.md`. The rest of the
  index is `/research`'s.
- **Don't build a vector index.** Matching here is exact overlap on authored
  tags + link-graph traversal, not similarity. Recall-by-meaning is
  `module.ingest` / the kernel (see `research-rules.md` boundary section).

## When NOT to use this skill

- **Authoring edges** (adding `@`/`[[ ]]`/`related`/tags) → `/link`.
- **Creating, logging, or concluding a topic** → `/research`.
- **Similarity recall over facts** → `module.ingest` / `memory_search`.

## What "done" looks like for an /xref session

The user leaves knowing the shape of the graph: what points at a topic
(back-references), what *could* connect that isn't linked yet (tag-match
discovery), where the graph is broken (danglers/mismatches), or with a
refreshed map in `INDEX.md`. The deliverable is the answer (or the staged
map), grounded in cited file locations.
