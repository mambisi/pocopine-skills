---
name: design-to-app
description: >-
  Use when turning a Claude Design (or any mockup/design) into a well-structured pocopine app — choosing app architecture (a store plus layout/leaf components), translating inline styles into Pine Stylekit, wiring icons and resizable regions, and verifying the result.
---

## What this is

A playbook for going from a **design** (a Claude Design `claude_design` MCP import,
a Figma export, or any mockup) to a **well-shaped pocopine app**. It is about
*structure and translation*, not any one feature — it ties together
`reactivity-and-stores`, `pocopine-components`, `pine-stylekit`, `pine-icons`, and
`slots-and-composition`. Read those for the details; read this to decide the shape.

The anti-pattern it exists to prevent: porting a design as **one giant component
with a `rebuild()` that computes every inline-style string**. That compiles and
renders, but it is unmaintainable and fights the framework. Decompose instead.

## When to use

- Importing a design / mockup and deciding how to lay the code out.
- An app has outgrown a single component and needs an architecture.
- Translating a richly-styled HTML/inline-style prototype into Stylekit utilities.

## Key API / syntax

**1 — State: one `#[store]` singleton.** Shared state (theme, selection, lists,
…) lives in a `#[store(name = "x")]`; components read it as `$store.x.<field>` in
templates and mutate it from handlers via
`pocopine::store::<XStore>().update(|s| s.action(...))`. Keep *derived display
state* as plain fields recomputed by a `rebuild()` the store calls at the end of
every action (see `examples/keep/src/store/derived.rs`'s `rebuild_visible_notes`),
or as `#[computed]` fields. → `reactivity-and-stores`.

**2 — Components: layout vs leaf.**
- *Layout / shell* components compose children and read `$store`; they hold no
  state of their own (`examples/keep/.../grid_layout`).
- *Leaf* components are presentational: a `#[prop] item: XxxVm` struct passed with
  `pp-bind:item="row"` (or several scalar `#[prop]`s) plus thin handlers that
  forward to the store (`examples/keep/.../note_card`).

**3 — Layout participation: `display: contents`.** A component renders as its
custom-element host (`<my-thing>`) wrapping the template root. By default the host
is `display: inline`, which breaks flex/grid. Put
`#[component(display = "contents")]` on any component that is a *layout child* so
its inner root becomes the real flex/grid item.

**4 — Declare child tags with `uses`.** Every custom tag a template renders —
`<my-child>`, `<pine-icon>`, `<pine-splitter-*>` — must be listed in
`#[component(uses = [..])]` (RFC 049), or the macro errors.

**5 — Styling: Pine Stylekit + theme tokens.** Put the palette in `app.css`
`@theme` as `--color-*` tokens; theme variants are `[data-theme="…"]` overrides of
those tokens (utilities compile to `var(--color-NAME)`, so one `:data-theme` on
the root reskins everything). Convert *static* inline styles to utility classes;
toggle state with `:data-active="expr"` + `data-[active=true]:…` variants; keep
*only* genuinely runtime values (a per-row colour, a drag transform) as inline
`:style`. → `pine-stylekit`.

**6 — Reuse Pine UI.** Icons → `<pine-icon name="…">` + `register_icons!`
(`pine-icons`). Resizable regions → `<pine-splitter-group>` / `-panel` /
`-resize-handle` from `pine-ui` (`crates/pine/src/splitter`). Prefer existing
components over hand-rolling drag/pointer logic.

## Examples

Suggested project layout (mirrors `examples/keep`):

```
src/
├── lib.rs              # module tree
├── app.rs              # #[wasm_bindgen(start)] main(): register_icons! + store + components
├── model.rs            # raw structs / domain types
├── store/              # the #[store] + view-models + rebuild() + actions + seed
└── components/<area>/  # <Name>.poco + mod.rs, grouped by region (shell, sidebar, …)
```

Leaf-from-the-store pattern (handler forwards to the store):

```rust
// components/sidebar/mod.rs
#[derive(Default, Serialize, Deserialize)]
#[component(display = "contents", uses = [PineIcon])]
pub struct ChannelList {}

#[handlers]
impl ChannelList {
    pub fn select(&mut self, id: String) {
        pocopine::store::<AppStore>().update(move |s| s.select_channel(id));
    }
}
```

```html
<!-- ChannelList.poco — state via :data-* + variant, icon via <pine-icon> -->
<template pp-for="ch in $store.app.channels" pp-key="ch.id">
  <div @click="select(ch.id)" :data-active="ch.id == $store.app.active"
       class="flex items-center gap-2 px-2 py-1 rounded-md hover:bg-hv
              data-[active=true]:bg-hv data-[active=true]:text-t1">
    <pine-icon name="hash" size="14"></pine-icon>
    <span class="truncate">{{ ch.name }}</span>
  </div>
</template>
```

## Gotchas

- **`text-[#hex]` is parsed as font-size, not colour** (Stylekit). Use a
  `--color-NAME` token + `text-NAME`; `bg-`/`border-[#hex]` are fine. See
  `pine-stylekit`.
- **Only statically-literal class names ship** — never concatenate class names;
  drive state with `:data-*` + `data-[…]:` variants.
- **Templates read; Rust computes** — no arithmetic / method calls in `{{ }}` or
  directive expressions. Parsing, formatting, colour lookups, percentages, etc.
  belong in Rust (`#[computed]` or the store `rebuild()`). See `poco-expressions`.
- **Forgetting `display: contents`** on a layout child collapses the layout (the
  inline host becomes the flex/grid item instead of your styled root).
- **Forgetting `uses`** for `<pine-icon>` / child tags is a compile error, not a
  silent miss.
- **`just check` ≠ correct.** It only proves compilation; Stylekit / theme /
  layout bugs surface only at runtime. Serve and screenshot the running app (e.g.
  headless `google-chrome` via `playwright`), flipping themes, before declaring
  done.

## References

- **Architecture model:** `examples/keep` — single `#[store]` (`src/store/`),
  derived view rows (`store/derived.rs`), layout shells (`components/grid_layout`,
  `components/list_detail`), leaf cards (`components/note_card`).
- **Stylekit theming:** `examples/file-browser/app.css` (`@theme --color-*` +
  `[data-theme="dark"]` overrides) and its `data-[active=true]:` variants.
- **Pine UI:** `crates/pine/src/splitter` (resizable panels),
  `crates/pine-icons` (`register_icons!`, `<pine-icon>`).
- **Sibling skills:** `reactivity-and-stores`, `pocopine-components`,
  `slots-and-composition`, `pine-stylekit`, `pine-icons`.
