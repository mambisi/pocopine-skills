# Contributing skills

This repo is a registry of Claude Code agent skills for the pocopine framework.
Each skill lives at `<slug>/SKILL.md`; `skills.json` is the machine-readable
index used by the `pocopine skills` CLI + MCP (see `DESIGN.md`).

## Add or edit a skill

1. Create `<slug>/SKILL.md` (kebab-case slug). Use this frontmatter:

   ```markdown
   ---
   name: <slug>
   description: >-
     Use when <the trigger — what task makes this skill relevant>
   ---

   ## What this is
   ...
   ## When to use
   ...
   ## Key API / syntax
   ...
   ## Examples
   <real snippets, with file-path citations into the pocopine source>
   ## Gotchas
   ...
   ## References
   <crate paths, RFC numbers, doc files>
   ```

2. **Ground every claim in the framework source.** Copy real snippets from
   `crates/…` and `examples/…`; cite paths. Don't invent API names. If you're
   fixing drift, say what changed (e.g. an RFC that renamed an attribute).

3. Keep the `description` a single line starting with **"Use when"** — it's the
   trigger text agents match on.

4. Regenerate the index so `skills.json` + the README table stay in sync, then
   commit both with your skill:

   ```bash
   node scripts/gen.js   # (the generator that reads each SKILL.md frontmatter)
   ```

## Review bar

- Accurate against the current framework (no stale/removed directives, correct
  macro/attribute names).
- Focused: one feature area per skill; link sibling skills rather than repeat.
- Examples compile (or are copied verbatim from code that does).

Open a PR; CI validates that every `<slug>/SKILL.md` has valid frontmatter and
that `skills.json` matches the tree.
