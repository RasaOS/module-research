# CHANGELOG — `rasa.module.research`

Reverse-chronological. Each entry is a version bump.

---

## 0.1.3 — 2026-07-09

### Element identity layer (canon SA-025)

- Added `rasa.identity` ("the RasaOS module for research-topic tracking and cross-referencing"); `bin/init` generates `.claude/rasa-identity.md` from it every install + stamps project-owned `.claude/rasa-deployment.md`; ships `/whoami`; CLAUDE.md "Who you are" header.

## 0.1.2 — 2026-07-09

### Added generic `/sync` + `/promote` + `/kit`-aware `bin/init` (canon SA-024)

- `bin/init` now clones the Element source into `<project>/kit/<element>/`; `/sync` smart-pulls upstream, `/promote` smart-pushes local edits back upstream (both directory-mirror → installed into consumers).

## 0.1.1 — 2026-07-09

### `parent_kind` → `[domain, tenant]` (canon SA-023)

- The `orchestrator` kind was folded into `tenant`; this module now mounts into a tenant or a domain (`requires.parent_kind: ["domain", "tenant"]`, was `["domain", "orchestrator"]`).

## 0.1.0 — 2026-07-09

Initial ship — a **full working module** (greenfield, author-requested; not a
distillation, so not gated behind a second consumer like `module.tasks`/`.notes`).

### Two capabilities

- **Track research topics** — a standard `research/<slug>/` folder per topic
  (README overview + `log` + `findings` + `sources` + `open-questions`), an
  `open→active⇄dormant→concluded→archived` lifecycle, and a regenerable
  `research/INDEX.md` registry.
- **Cross-reference / match** — a layered link system: `@path` context pointers
  (Layer 1), `[[topic-slug]]` wikilinks for the bidirectional topic graph
  (Layer 2), and `tags:`/`related:` frontmatter for structured matching
  (Layer 3). `/xref` computes back-references (reverse `[[...]]`) and tag-match
  discovery (related topics nobody explicitly linked).

### Ships

- **Spine** — `content/research-rules.md` → `.claude/research-rules.md`
  (file-replace): the topic model, structure, lifecycle, the three-layer
  cross-reference/matching model, INDEX conventions, the conclude/promotion
  flow, and the boundary against the memory siblings.
- **Three skills** — `content/skills/{research,link,xref}/SKILL.md` →
  `.claude/skills/` (directory-mirror): `/research` (topic lifecycle), `/link`
  (author cross-references), `/xref` (resolve + discover/match).
- **Per-topic templates** — `content/topic-template/` →
  `.claude/research-topic-template/` (directory-mirror): the five skeleton
  files `/research new` instantiates.
- **Seam** — `seed/research-canon.md.template` → `.claude/research-canon.md`
  (skip-if-exists): the per-project research root + tag taxonomy +
  `promotion_target` + `promotion_criteria`. `/research conclude` hard-stops
  until `promotion_target` is set.
- **Index** — `seed/research/INDEX.md.template` → `research/INDEX.md`
  (skip-if-exists): the topic registry + the `XREF MAP` block.
- **Author docs** — `content/README.md` + `content/BUILD_PLAN.md` (opt-in).
- Canon-required files: Apache-2.0 LICENSE, `bin/init`, `bin/check-manifest`,
  SHA-stamped lockfile template, `.gitignore`.

### Notes

- Scaffolded via `bin/new-element module research`; the domain-core universal
  starter layer (codify/onboard/handoff skills, rules, agents, templates,
  SHAPE.md, check-shape) was stripped to match the lean `module.tasks`/`.notes`
  shape.
- Soft cross-references to `module.notes` (the primary promotion target),
  `module.tasks`, `module.ingest` (kernel similarity-recall — a *different*
  primitive), and `module.jobs`. None hardened into `requires.elements[]`.
- `bin/check-manifest` GREEN; `bin/init` smoke-tested into a scratch target.
- Built locally; not yet pushed. Target repo: `RasaOS/module-research`.
