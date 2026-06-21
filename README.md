# pocopine skills

A registry of [Claude Code](https://docs.anthropic.com/en/docs/claude-code) **agent skills** for the
[pocopine](https://github.com/mambisi/pocopine) framework — one `SKILL.md` per feature area, each
teaching an agent how to use that part of the framework correctly (when to use it, key API/syntax,
real examples, gotchas, references).

## Using these skills

**As a submodule** (pins the full set into a project):

```bash
git submodule add https://github.com/mambisi/pocopine-skills.git .claude/skills
```

Claude Code auto-discovers `.claude/skills/<name>/SKILL.md`, so the skills become available with no
further setup.

**On demand** (planned, see [`DESIGN.md`](./DESIGN.md)):

```bash
pocopine skills list                 # browse the registry
pocopine skills install routing      # copy one skill into ./.claude/skills/
```

The same operations are exposed over MCP so an agent can install skills as it needs them.

## Contributing

New skills and fixes are welcome — see [`CONTRIBUTING.md`](./CONTRIBUTING.md). `skills.json` is the
machine-readable index (regenerated from each skill's frontmatter).

> Skills here are drafts authored against the framework source; verify against the code before relying
> on a detail. PRs that correct or extend them are the point of this repo.

## Index

### Core authoring

| Skill | Use when |
| --- | --- |
| `design-to-app` | turning a Claude Design (or any mockup/design) into a well-structured pocopine app — choosing app architecture (a store plus layout/leaf components), translating inline styles into Pine Stylekit, wiring icons and resizable regions, and verifying the result. |
| `poco-directives` | authoring pocopine template directives (pp-*), understanding directive syntax, modifiers, args, host constraints, or migrating from removed directives (pp-init/pp-cloak/pp-data) |
| `poco-expressions` | writing or debugging {{expr}} interpolation and pine-expr expressions in .poco templates (paths, operators, ternary, calls, assignment, magic variables like $index/$event/$store/$route). |
| `poco-templates` | writing or debugging pocopine .poco template syntax, single-root rules, component tags, SVG namespace support, or template compilation in the pocopine framework. |
| `pocopine-components` | defining, structuring, or debugging pocopine Components — the #[component] and #[handlers] macros, #[prop]/#[model] fields, templates, lifecycle hooks, and component composition patterns. |
| `reactivity-and-stores` | building state management and reactivity in pocopine — understand the reactive model, App stores, and provide/inject context |
| `slots-and-composition` | building or integrating components with default/named slots, scoped slots with pp-let, pp-as polymorphic rendering, or multi-component composition patterns in pocopine |

### UI & styling

| Skill | Use when |
| --- | --- |
| `animation-and-motion` | building enter/leave transitions, layout animations, stagger effects, or spring-physics motion in pocopine components |
| `interaction-utilities` | implementing keyboard navigation, floating positioning, element observation, focus management, scroll locking, or accessibility wiring in pocopine components |
| `pine-charts` | building SVG-first, unstyled, accessible charts in pocopine with line, area, bar, scatter, pie, or custom layered visualizations |
| `pine-icons` | working with the pocopine Pine icons feature — the `icon!` proc macro for compile-time Rust SVG embedding, or the `<pine-icon>` template primitive with `register_icons!` for tree-shaking-friendly template rendering. |
| `pine-richtext` | building Rust/WASM rich-text editors with the Pine document model, schema setup, markdown round-trips, or table extensions |
| `pine-stylekit` | building styles with Pine Stylekit, the Pocopine-native utility-CSS compiler, or working with @theme tokens and CSS generation in Rust/WASM projects |
| `scoped-styles` | authoring component stylesheets with automatic CSS scoping to a single component. |

### Data & backend

| Skill | Use when |
| --- | --- |
| `auth` | implementing authentication in pocopine apps — JWT verification, credentials, OAuth providers, session management, or guards |
| `background-jobs` | defining background jobs, enqueueing work, configuring workers, or troubleshooting job execution in pocopine apps |
| `observability` | setting up tracing, logging, analytics, or observability events in a pocopine application—both frontend and backend. |
| `routing` | building SPA routes, implementing route guards/loaders, configuring pp-route links, or handling route navigation in pocopine. |
| `server-functions` | defining typed async server functions with `#[server]` macro, implementing access policies via guards, and calling them from the wasm client with `dispatch!` for async state updates. |
| `storage` | integrating object storage (S3/GCS/Azure) with resumable uploads, presigned/multipart/sequential transfer strategies, or browser-based file uploads |
| `sync-and-live` | building offline-first data sync features or live invalidation in Pocopine apps — implementing sync streams, mutations, subscriptions, or multi-tenant filtered queries with local persistence. |

### Tooling & ops

| Skill | Use when |
| --- | --- |
| `client-modules` | setting up or working with Pocopine's managed .client.ts modules for importing npm SDKs (Firebase, analytics, etc.) with typed Rust facades |
| `deploy` | deploying a pocopine full-stack or static app to Fly.io, Railway, Render, or other hosts using the RFC 080 deploy contract |
| `plugins` | installing app or server plugins—wiring observability, auth, live queries, or other optional integrations into a pocopine app. |
| `pocopine-cli` | working with the pocopine-cli: build, dev watch, run, deploy, doctor, env, js, stylekit, lsp commands |

### Framework internals

| Skill | Use when |
| --- | --- |
| `codec-crypto` | adding or touching any hashing, checksums, base64, percent-encoding / URL escaping, or request signing (HMAC, Azure Shared-Key / SAS, token hashing) in the pocopine workspace. Routes the work through the shared pocopine-crypto and pocopine-codec crates instead of pulling in sha2 / md-5 / crc32c / hmac / base64 / percent-encoding directly or hand-rolling encoders. |
