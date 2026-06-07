# Design — `pocopine skills` CLI + MCP installer

This registry is consumed two ways: **pinned** (git submodule, for the core repo
and the starter) and **on demand** (a `pocopine skills` CLI + an MCP server, for
any other project). This document specs the on-demand path.

## Goals

- `pocopine skills install <name>` drops a skill into a project's
  `.claude/skills/` so Claude Code picks it up — no submodule required.
- An **MCP server** exposes the same operations as tools, so an agent can
  discover and install skills *itself* mid-task ("I need the routing skill →
  install it").
- Community contribution stays a plain PR to this repo (`CONTRIBUTING.md`).

## CLI

A `skills` subcommand on the pocopine CLI (`crates/pocopine-cli`):

```
pocopine skills list                 # print the registry index (from skills.json)
pocopine skills search <query>       # fuzzy match name + description
pocopine skills install <name>...    # copy <name>/ into ./.claude/skills/<name>/
pocopine skills update [<name>]      # re-pull installed skills from the registry
pocopine skills remove <name>        # delete ./.claude/skills/<name>/
pocopine skills mcp                  # run the MCP server over stdio (see below)
```

**Source resolution.** Default registry = `https://github.com/mambisi/pocopine-skills`
(overridable via `--registry <git-url>` or `[package.metadata.pocopine.skills] registry = …`).
The CLI keeps a shallow cache under `~/.pocopine/skills/` (clone once, `git pull`
on `update`), reads `skills.json` for the index, and copies the requested
`<slug>/` directory into the target project's `.claude/skills/`. A private
registry just relies on the user's existing git credentials.

**Install semantics.** Copy (not submodule) so the skill is editable in-project
and survives offline; record provenance in `.claude/skills/<slug>/.source` (the
registry URL + commit) so `update` knows what to re-pull and can detect local
edits.

## MCP server

`pocopine skills mcp` runs a stdio MCP server (built on the Rust MCP SDK)
exposing:

| Tool | Purpose |
| --- | --- |
| `list_skills` | return the registry index (name, description, category) |
| `search_skills(query)` | ranked matches over name + description |
| `read_skill(name)` | return a skill's markdown without installing (preview) |
| `install_skill(name)` | write `<slug>/SKILL.md` into the project `.claude/skills/` |

Registered in a project's MCP config so the agent can call `install_skill`
on demand. `read_skill` lets the agent consult a skill without committing it.

## Contribution flow

PR to this repo → CI validates frontmatter + that `skills.json` matches the tree
→ merge. Installs/updates pull from the default branch (or a pinned tag once the
registry versions releases).

## Open questions

1. **Update vs local edits** — `update` should 3-way merge or refuse on a dirty
   skill; the `.source` marker enables detection.
2. **Versioning** — tag the registry and let `install` pin a tag, or always track
   the tip? Start with tip; add tags when skills stabilize.
3. **Private registries** — git credentials cover the owner; a hosted index
   (read-only HTTP) could serve `list`/`search` without clone for public use.
4. **Dedup with the submodule** — in the core repo + starter the submodule is the
   source of truth; the CLI is for projects that don't vendor the full set.

Status: design only. Implement behind a `skills` subcommand in `pocopine-cli`;
the MCP server can reuse the same registry-resolution code.
