# CLAUDE.md — `rasa.module.research`

> **Who you are (SA-025).** `rasa.module.research` — the RasaOS module for research-topic tracking and cross-referencing. Substrate: **RasaOS**; role: **module**. On install `bin/init` renders this into `.claude/rasa-identity.md`; `/whoami` composes the full identity with the project's deployment layer.


Per-repo working contract for Claude sessions opened inside this folder.
Extends `~/.claude/CLAUDE.md` and the workspace `rasa.tenant.rasaos` tenant
contract; does not override them. (Referenced by identity, not filesystem
path.)

## What you are when you're in this folder

You are working on **`rasa.module.research`** — a `module`-kind Element: a
portable research workspace that (1) tracks research topics as structured
folders and (2) cross-references/matches them with a layered link system.
Sibling to `module.tasks`, `module.notes` (the promotion target for a concluded
finding), and `module.ingest` (which wraps the kernel's *other* memory
primitive).

A `module` (canon Spec §6) is "a focused capability that extends a domain or
orchestrator, mountable into one or more parents." Parents pull it via their
own `requires.elements[]`.

## The load-bearing ideas

**1. Two capabilities, one spine.** Track (folder-per-topic + lifecycle +
index) and cross-reference/match (three link layers + back-references +
tag-discovery). Both live in `content/research-rules.md`. Don't let one eclipse
the other — the cross-reference system is half the reason the module exists.

**2. The three cross-reference layers are distinct and compose.** `@path`
(content-level "read this file"), `[[slug]]` (the bidirectional topic graph),
`tags:`/`related:` (structured matching). Each does one job. Don't collapse
them into one syntax; the design decision (chosen by the author) is that all
three coexist. `/link` writes them; `/xref` reads/computes over them
(back-references and tag-matches are *computed*, not stored).

**3. The seam is load-bearing.** The one per-project thing — research root, tag
taxonomy, and `promotion_target` — is the project-owned
`.claude/research-canon.md` (seeded skip-if-exists). `/research conclude`
hard-stops if `promotion_target` is unfilled. `content/` references it; it
hardcodes none of it.

**4. The boundary against the memory siblings is load-bearing.** This module is
an *ongoing structured investigation* (a linked corpus a person reads).
`module.notes` is *ordered ratified decisions* (a concluded topic promotes
*into* it). Kernel similarity-recall (`module.ingest`) is *recall-by-meaning*.
If you find yourself adding vector search or similarity recall **here**, stop —
that's the kernel's job. The cross-references to all siblings are soft.

**5. Greenfield, not a distillation.** Unlike `module.tasks`/`module.notes`
(extract-after-proof), this was built from scratch on author request, so v0.1.0
ships the working skills — it is not gated behind a second consumer. (Hardening
after a first real consumer is still the v0.2.0 plan; see `BUILD_PLAN.md`.)

## Current status — full working v0.1.0

Ships the spine (`content/research-rules.md`), the seam
(`seed/research-canon.md.template`), the three skills (`/research`, `/link`,
`/xref`), the per-topic templates (`content/topic-template/`), and the seeded
index (`seed/research/INDEX.md.template`). `bin/check-manifest` GREEN;
`bin/init` smoke-tested.

## Source of truth

- **Canon** (`rasa.tenant.rasaos` workspace `canon/`) — authoritative. Spec §6
  defines the `module` kind; ELEMENT_CONTRACT.md §7 the install policies.
- **`content/research-rules.md`** — the installed spine (the discipline).
- **`content/BUILD_PLAN.md`** — the spec + the three-skill contract + the
  memory-sibling boundary + the version plan.
- **`rasa.json`** — the formal declaration + install manifest.

## Don'ts

- **You are NOT the template.** If this contract ever describes `domain-core`
  (or any other Element), the template-CLAUDE.md drift class is back — flag it.
- **Don't build the kernel's memory here** (vector search, similarity recall).
  Soft-reference `module.ingest`; never harden it into `requires.elements[]`.
- **Don't collapse the three link layers** into one syntax, and don't drop the
  cross-reference/matching capability in favor of just topic-tracking.
- **Don't let `/research conclude` promote on its own authority.** It stages;
  the human commits; it hard-stops without a `promotion_target`.
- **Don't invent tag taxonomy in the skills.** Tags come from the seam.
- **Don't `bin/init` this Element into itself.** `content/` is the source.
- **Don't push from the Cowork sandbox.** Local commit + tag only; the author
  pushes from their machine (workspace rule).

## How a version bump works

- **Patch** — wording/template fix. **Minor (→0.2.0)** — seam hardening after a
  first consumer, `/xref` output polish, a new capability, an `_archive/`
  convention. **Major (→1.0.0)** — seam format + topic structure + link model
  locked after two real verticals run the pipeline unchanged.

Each bump: edit `VERSION` + `rasa.json#version`, write a CHANGELOG entry, run
`bin/check-manifest`, commit + tag `v<version>`. Update the workspace
`elements/REGISTRY.md` + `elements/CHANGELOG.md` (track #2).
