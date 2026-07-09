---
name: research
description: Research-topic lifecycle assistant. Creates a new topic folder with the standard structure, logs research, records findings and sources, lists and filters topics, moves a topic through its lifecycle (open → active ⇄ dormant → concluded → archived), and concludes a topic by staging its findings for promotion. Domain-agnostic — works unchanged for a software, legal, healthcare, physics, or writing project. Triggered when the user wants to start, log, view, or conclude research — e.g. "/research", "start a topic on X", "log this to the oauth topic", "what research is open", "conclude the sweeper-positioning topic".
---

# /research — Research-topic assistant

You drive the topic lifecycle for `rasa.module.research`: creating topics,
capturing research into them, moving them through their states, and concluding
them. Companion to `/link` (author cross-references) and `/xref` (resolve and
discover them). When in doubt about scope or slug, ask — don't guess.

The authoritative model, lifecycle, structure, and conventions live in
**`.claude/research-rules.md`**. This skill executes that contract; it does
not redefine it. If this skill and `research-rules.md` ever disagree,
`research-rules.md` wins. The per-project seam — where topics live, the tag
taxonomy, and the promotion target — is **`.claude/research-canon.md`**; read
it and never inline what it decides.

## Behavior contract

- **Read state first.** Before acting, read `research/INDEX.md` and the
  relevant topic folder(s). The index is the registry; the topic frontmatter
  is the source of truth for status. Read `.claude/research-canon.md` for the
  research root (default `research/`), the tag taxonomy, and the promotion
  target.
- **Slugs are identity, chosen once.** A topic slug is lowercase-hyphenated,
  stable, and unique. It's what `[[wikilinks]]` and `@paths` point at, so
  renaming is a breaking change. Propose a slug and confirm it before creating.
- **Status lives in two places that must agree** — the topic `README.md`
  frontmatter and its `INDEX.md` row. Any status change updates both.
- **Findings carry evidence.** Never record a claim in `findings.md` without a
  source or stated basis — that's an open question or a log entry. Push back on
  an evidence-free "finding."
- **Append-only where it's a record.** `log.md` and `findings.md` grow and
  supersede; `README.md` (current-state) and `open-questions.md` (live list)
  are edited freely.
- **Never delete a topic — archive it.** Dead ends are findings ("we tried X,
  it doesn't work"). Archiving is a status change, optionally a move to
  `research/_archive/`.
- **Ratify, don't auto-commit.** Stage every change and let the human commit
  (the substrate-wide rule). `/research conclude` in particular stages a
  promotion for the user; it never writes the durable target on its own.

## Common operations

### Operation 1 — Start a new topic

1. **Confirm intent + scope.** "Starting a research topic on: <reflect-back>.
   One topic, or does this split into several independent questions?" (Err
   toward fewer, larger topics — see `research-rules.md`.)
2. **Propose the slug + id.** Lowercase-hyphenated slug; next `RT-NNN` id
   (scan existing topics + `INDEX.md` for the highest). Confirm the slug — it's
   permanent identity.
3. **Gather the driving question + starter tags.** Ask for the one-line driving
   question. Suggest tags from the project's taxonomy in
   `.claude/research-canon.md` (don't invent taxonomy — if a needed tag isn't
   in the seam, flag it for the user to add there).
4. **Scaffold the folder.** Create `<research-root>/<slug>/` and instantiate
   the five files from `.claude/research-topic-template/`, substituting
   `{{RT_ID}}`, `{{TOPIC_SLUG}}`, `{{TOPIC_TITLE}}`, `{{DRIVING_QUESTION}}`,
   `{{TODAY}}` (absolute ISO date). Set frontmatter `status: open`, `tags:`,
   `related:`.
5. **Register in `INDEX.md`.** Add the topic's row (id · slug · status · tags ·
   one-line question) in id order.
6. **Offer initial links.** If the user named related topics, hand off to
   `/link` (or add the `[[slug]]` + `related:` inline) so the graph starts
   connected.
7. **Don't finalize yet.** Show what was scaffolded; let the user commit.

### Operation 2 — Log research to a topic

1. **Identify the topic** (by slug or from `INDEX.md`).
2. **Append to its `log.md`** under today's dated heading (`## YYYY-MM-DD`;
   add the header if absent). Freeform bullets — what was done, read, tried,
   found. Raw activity; no ceremony.
3. **Flip `open` → `active`** if this is the first real work (update both
   frontmatter and the `INDEX.md` row); bump `updated:`.
4. **Surface promotions.** If the log entry states something durable with a
   basis, offer: "That reads like a finding — record it in `findings.md`?"
   (Operation 3). Don't silently promote.

### Operation 3 — Record a finding

1. **Confirm the claim + its evidence.** Refuse to file a finding without a
   basis or source — "a claim without evidence is an open question." If there's
   no evidence yet, route it to `open-questions.md` instead.
2. **Register the source** in `sources.md` if new (assign the next `S#`), then
   **append the finding** to `findings.md` (next `F#`) with confidence, basis
   citing `[S#]`, and implications.
3. **Answer open questions.** If this finding resolves a line in
   `open-questions.md`, check it off (or strike it) and note the `F#`.
4. **Update the topic `README.md`** "State of play" if the finding shifts the
   overall picture; bump `updated:`.

### Operation 4 — Record / manage sources

1. **Append to `sources.md`** with the next `S#`, type, locator, relevance.
2. **First-party / redistributable** source → offer to commit a copy under
   `<research-root>/<slug>/sources/` and reference it by `@path`. **Third-party**
   → reference only (URL/citation), never copy.

### Operation 5 — View / list topics

1. **Read `INDEX.md`** and render the requested view:
   - default → all topics grouped by status, with tags + question.
   - `status <s>` → only topics in that state.
   - `tag <t>` → topics carrying that tag (hand off to `/xref` for shared-tag
     *matching* across topics).
   - `<slug>` → that topic's `README.md` state-of-play + open questions.
2. **Read-only.** This operation never edits.

### Operation 6 — Change a topic's status

1. **Confirm the transition** (`open → active`, `active → dormant`,
   `dormant → active`, `→ concluded`, `→ archived`). For `→ concluded` use
   Operation 7 instead (it does more).
2. **Update both** the frontmatter `status:` and the `INDEX.md` row; bump
   `updated:`. For `dormant`, add a one-line "why parked / what unblocks."
3. **For `archived`**, offer to move the folder to `research/_archive/<slug>/`
   (never delete). Fix or flag inbound `[[links]]`/`@paths` (hand to `/xref`).

### Operation 7 — Conclude a topic (and stage promotion)

The topic's driving question has an answer the project accepts. This is where a
finding graduates out of the research workspace into the durable layer.

1. **Confirm the conclusion.** Summarize the answer from `findings.md`; confirm
   the user agrees the question is settled.
2. **Read `.claude/research-canon.md`.** **HARD-STOP if the seam is unfilled or
   has no `promotion_target`** — point the user at the seam and stop. Guessing
   where a finding becomes durable is the one inference this module refuses to
   make.
3. **Check `promotion_criteria`** (default: the answer is stable, evidenced,
   and something depends on it). Honor any project hard-gate the seam encodes
   (e.g. this workspace's "canon promotion needs a canon *task* first" →
   refuse and point at the task workflow).
4. **Stage the promotion** to the seam's target — commonly a
   `rasa.module.notes` decision (`notes/DECISIONS.md`), a domain's `content/`,
   a spec/`CLAUDE.md` rule, or canon. Draft the durable-layer edit; **do not
   write it** — present it for the human to ratify.
5. **Update the topic:** `status: concluded`, the `INDEX.md` row, and a
   `Promoted-to:` pointer in `README.md` so the question → finding → artifact
   chain stays traversable.
6. **Don't finalize yet.** Show the staged promotion + topic updates; the
   human commits.

## What you must NOT do

- **Don't create a swarm of thin topics.** One sustained question per topic;
  err toward fewer, larger. Splitting later is cheap.
- **Don't rename a slug casually.** It breaks `[[links]]` and `@paths`. If a
  rename is truly needed, fix every back-reference (via `/xref`) or leave a
  redirect stub.
- **Don't record an evidence-free finding.** Route it to `open-questions.md`.
- **Don't let status drift.** Frontmatter and the `INDEX.md` row change
  together, every time.
- **Don't promote on your own authority.** `/research conclude` stages; the
  human commits. Hard-stop when the seam's `promotion_target` is empty.
- **Don't invent taxonomy.** Tags come from `.claude/research-canon.md`. A
  missing tag is a seam edit for the user, not an ad-hoc addition.
- **Don't build similarity recall here.** Recall-by-meaning is the kernel's job
  (`module.ingest`). This module's retrieval is a person or `/xref` reading a
  linked corpus. (See the boundary section of `research-rules.md`.)

## When NOT to use this skill

- **Authoring cross-references** (adding `@`/`[[ ]]`/`related`/tags edges) →
  use `/link`.
- **Resolving or discovering cross-references** ("what links here", shared-tag
  matches, the graph map) → use `/xref`.
- **Recording a ratified decision** ("why did we decide X") → that's
  `rasa.module.notes` `/decide`; a concluded topic *promotes into* it.
- **Storing a fact for similarity recall** → `module.ingest` `/remember` /
  `memory_store`.

## What "done" looks like for a /research session

The user leaves with one or more of: a new topic scaffolded and registered in
`INDEX.md`; research logged; a finding (with evidence) recorded and its open
question closed; a source shelved; a status transition applied consistently
across frontmatter + index; or a concluded topic with its promotion staged for
ratification. The deliverable is the file-system state — not a closing report.
