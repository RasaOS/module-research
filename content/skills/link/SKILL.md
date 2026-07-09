---
name: link
description: Cross-reference authoring assistant for research topics. Adds and maintains the three layers of linking — @path context pointers (pull a file into context), [[topic-slug]] wikilinks (the bidirectional topic graph), and tags/related frontmatter (structured matching). Keeps the [[...]] prose links and the `related:` frontmatter in sync, and validates that link targets exist. Triggered when the user wants to connect research — e.g. "/link", "link the oauth topic to jwt-rotation", "tag this topic security", "add a pointer from findings to the device-flow paper", "cross-reference these two topics".
---

# /link — Cross-reference authoring

You author the connections between research topics for
`rasa.module.research` — the second thing the module exists to do. Companion
to `/research` (which tracks the topics) and `/xref` (which *resolves and
discovers* over the links you author here). `/link` is the **write** side of
cross-referencing; `/xref` is the **read/match** side.

The authoritative link model — the three layers and when to use each — lives in
**`.claude/research-rules.md`** (§ "Capability 2 — cross-reference &
matching"). This skill executes that model; read it if unsure which layer
fits. Tags come from the project taxonomy in **`.claude/research-canon.md`** —
never invent tags outside it.

## The three layers (what you're authoring)

| Layer | Syntax | Job | Lives in |
|---|---|---|---|
| **1 — context pointer** | `@research/<slug>/<file>` | "to understand this, also read that file" (content-level, directional) | inline in `README.md` / `findings.md` / `log.md` |
| **2 — topic graph** | `[[<slug>]]` | "this topic relates to that topic" (bidirectional edge) | inline in prose + mirrored in `related:` |
| **3 — matching** | `tags: [...]`, `related: [...]` | structured fields the matcher queries | topic `README.md` frontmatter |

**Rule of thumb:** `@` for content you must read, `[[ ]]` + `related:` for the
topic graph, `tags:` for discovery. Author generously — the matcher (`/xref`)
only helps when the edges exist.

## Behavior contract

- **Read both endpoints first.** Before linking topic A to topic B, read both
  `README.md` files so the link is real and the direction is right. Never link
  to a slug you haven't confirmed exists (that's a *dangling* link — allowed,
  but flag it as "creates a dangling link to [[X]] — create that topic?").
- **Keep Layer 2 and Layer 3 in sync.** Every `[[slug]]` you add in a topic's
  prose gets its slug added to that topic's `related:` frontmatter, and vice
  versa. They are the same edge recorded two ways; `/xref` flags any mismatch,
  so don't create one.
- **Point `@path` at the file that matters.** When it's the *contents* you
  need, link the specific file (`@research/<slug>/findings.md`), not just the
  folder. Paths are workspace-relative and must resolve.
- **Tags come from the seam.** Use only taxonomy terms in
  `.claude/research-canon.md`. If the user wants a tag that isn't there, stop
  and offer to add it to the seam first — don't sprinkle ad-hoc tags that the
  matcher can't rely on.
- **Bump `updated:`** on any topic whose frontmatter or body you touch.
- **Don't finalize.** Stage edits; the human commits.

## Common operations

### Operation 1 — Link two topics (Layer 2 + 3)

1. **Confirm the pair + direction.** "Linking [[A]] ↔ [[B]] — mutual, or does
   only A reference B?" Most topic relationships are mutual; author both
   directions unless the user says otherwise.
2. **Read both `README.md`s.** Confirm both slugs exist. If one doesn't, offer
   to `/research new` it (or record the intentional dangling link).
3. **Add the prose link.** Under "Related topics" in each topic's `README.md`,
   add `[[other-slug]] — <one line on how they relate>`.
4. **Mirror into frontmatter.** Add each slug to the other's `related:` list.
5. **Show the reconciled state**; don't finalize.

### Operation 2 — Add a context pointer (Layer 1)

1. **Identify the source location** (where the `@` pointer goes — a finding, a
   log entry, the topic README's "Key context") **and the target file** (what
   must be read).
2. **Confirm the target resolves** (workspace-relative path exists).
3. **Insert `@path`** inline with a short "why it matters here" note. Point at
   the specific file whose contents are needed.
4. **Don't finalize.**

### Operation 3 — Tag a topic (Layer 3)

1. **Read the taxonomy** in `.claude/research-canon.md`.
2. **Propose tags** from it that fit; confirm with the user. Flag any requested
   tag that isn't in the taxonomy → offer to add it to the seam first.
3. **Update `tags:`** in the topic `README.md` frontmatter; bump `updated:`.
4. **Note the matching payoff:** "run `/xref tag <t>` to see other topics that
   now match on this tag." Don't finalize.

### Operation 4 — Link research to a source or external file

For pointing a finding at the evidence behind it:

1. **Ensure the source is shelved** in the topic's `sources.md` (hand to
   `/research` Operation 4 if not) — it gets an `S#`.
2. **Cite it** from the finding in `findings.md` as `[S#]`, and, if the source
   is a committed local file, add an `@path` pointer to it.
3. **Don't finalize.**

### Operation 5 — Re-slug fix-up (repair links after a rename)

When a topic slug changed and links now dangle:

1. **Get the old → new slug** from the user.
2. **Hand off to `/xref`** to find every inbound `[[old-slug]]` and `@research/old-slug/...`
   across all topics + `INDEX.md`.
3. **Rewrite each** to the new slug (prose `[[...]]`, `related:` entries, and
   `@paths`). Offer to leave a redirect stub at the old slug if anything
   external may still point at it.
4. **Don't finalize.**

## What you must NOT do

- **Don't author a Layer-2 link on only one side.** `[[...]]` in prose and the
  `related:` frontmatter entry go together, both directions unless told
  otherwise. A one-sided edit is exactly the drift `/xref` will flag.
- **Don't link to an unconfirmed slug silently.** A dangling `[[link]]` is a
  deliberate "create this next" marker — surface it, don't hide it.
- **Don't invent tags.** Taxonomy is the seam's. Missing tag → seam edit first.
- **Don't write an `@path` that doesn't resolve.** Confirm the target exists.
- **Don't finalize the commit.** Stage; the human commits.

## When NOT to use this skill

- **Discovering links / matches** ("what points at this topic", shared-tag
  matches, the whole graph) → `/xref`.
- **Creating or concluding a topic** → `/research`.
- **Cross-referencing into a *decision*** (institutional memory) → that's
  `rasa.module.notes`; a concluded topic promotes into it via
  `/research conclude`.

## What "done" looks like for a /link session

The user leaves with new, consistent edges: `[[...]]` links and `related:`
frontmatter in sync across both endpoints, `@path` pointers that resolve, tags
drawn from the taxonomy — the graph richer and ready for `/xref` to traverse.
The deliverable is the file-system state, not a report.
