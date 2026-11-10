<div align="center">

# Origin Atlas

**ORIGIN** — a live routing atlas, scaffolded as the Proa `origin` reference
application. Enter any domain, IP, prefix, or ASN and watch one lookup walk
through DNS, addressing, and BGP on real RIPEstat data.

[Proa docs](https://proa.so/docs) ·
[Component catalog](https://ui.proa.so) ·
[Framework source](https://github.com/Proa-Labs/proa)

</div>

---

## Run it

```console
cp .env.example .env
proa dev --open
```

`proa build` produces the release binary; `proa test` runs the suite through
the linked island pipeline (plain `cargo test` rejects island binaries at
link time by design).

## How this app fits together

| Path | What lives there |
| --- | --- |
| `src/lib.rs` | Routes: the page, `/site/<host>.md`, `/lab/rows`, health |
| `src/loader.rs` | The RIPEstat `DataLoader`: request-scoped dedup + 60s TTL cache |
| `src/api.rs` | RIPEstat client with per-widget timeout budgets |
| `src/model.rs` | Curated dossiers, carrier names, collector locations |
| `src/components/` | One directory per component: `mod.rs`, `recipe.rs`, the component, and `md.rs` |
| `src/components/_proa/` | Vendored proa-ui foundation (owned source, edit like any file) |
| `src/layouts/root.rs` | Document shell: fonts, theme bootstrap, cache-busted assets |
| `styles/input.css` | Tailwind v4 entry: ORIGIN palette tokens, keyframes, responsive layer |
| `public/origin.js` | Progressive engine: RIS Live socket, feeds, render lab |
| `tests/golden/home.html` | Byte-for-byte pin of the server-rendered document |

## Editing guide (read this before changing anything, humans and agents alike)

1. **Every visible component renders twice.** HTML lives in the component
   file (`html_sync!` for sync, `html!` for async); the Markdown twin lives
   next to it in `md.rs` (`md_sync!`). If you change one surface, change the
   other — the golden test and `/site/<host>.md` will catch drift.
2. **Classes belong in `recipe.rs`.** Components use Tailwind utilities via
   recipe constants; run `proa dev` and the pipeline recompiles
   `public/styles.css` from `styles/input.css`. New colors go in the
   `@theme` block as tokens, not as hex literals in markup.
3. **Interactivity is an rsjs island only when it needs a typed browser
   API** (clipboard, signals, dialogs). Everything else is server-rendered
   or lives in `public/origin.js` as progressive enhancement.
4. **Data flows through the loader.** Panes request one shared
   `ripestat:dossier:<query>` key; the resolver collapses a whole render
   into one upstream fan-out and the TTL cache serves reloads. Never fetch
   from a component.
5. **The golden test is the contract.** After an intentional markup change:
   `PROA_UPDATE_GOLDEN=1 proa test --test atlas`, then re-run `proa test`.
6. **Before you finish:** `proa fmt`, `proa lint`, `proa build`. Intentional
   allocations need a `// proa-perf-ok: reason` marker; the lint fails the
   build on unannotated warnings.

## Full documentation for agents

Fetch these as plain text — they are written to be read by tooling:

- `https://proa.so/llms-full.txt` — the complete Proa framework docs
- `https://ui.proa.so/llms-full.txt` — the complete proa-ui component docs

To add a catalog component: `proa ui add <name>` vendors its source into
`src/components/<name>/` where you own it.

---

Package: `origin-atlas` · SSR contract pinned by `tests/golden/home.html`

## Dependency access

Proa dependencies resolve from crates.io. No GitHub credentials or Cargo registry token is required to build the generated project.
