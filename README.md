# Don't Panic! — A Pocket Guide to the Settled Systems

A cross-linked, filterable encyclopedia of the *Starfield* (Bethesda) universe — factions, worlds, cosmology, and the state of the game itself. Styled to echo bethesda.net/game/starfield.

**Live site:** https://fidgetrope.github.io/dont-panic-starfield/

## What makes this different from a normal wiki page

- **Every entry is addressable.** Click any card and the URL updates to `#entry-<id>` — copy that link straight to someone.
- **Real cross-referencing.** Entries carry an explicit `related` list, rendered as a "See also" row inside the detail view, so following the lore doesn't dead-end at one card.
- **Faceted filtering, not just search.** Filter by faction, system, and era at once — combinations sync to the URL (`?faction=crimson&systems=Kryx`), so a filtered view is itself a link you can share.

## Running it locally

This is a static site with no build step, but it fetches its content from `data/*.json` — that means it needs to be served over HTTP, not opened directly as a `file://` URL (browsers block `fetch()` of local files for security reasons). Any static server works:

```bash
python -m http.server 8000
```

Then open `http://localhost:8000`.

## Adding or editing content

All content lives in `data/*.json` — nothing is hand-typed into `index.html` itself. To add an entry:

1. Open the relevant file (`factions.json`, `locations.json`, `cosmology.json`, `updates.json`, `fieldnotes.json`, or `links.json`).
2. Copy the shape of a neighbouring entry and fill in your own `id` (kebab-case, must be unique across `factions.json` + `locations.json` + `cosmology.json`), `name`, `summary`, `body` (an array of paragraph strings), and any facets that apply (`system`, `faction`, `era`, `tags`).
3. Add the new entry's `id` to the `related` array of anything it should cross-link from, and add other entries' ids to *its* `related` array.
4. Reload — no rebuild required.

### Schema quick reference

| File | Facets used for filtering | Notes |
|---|---|---|
| `factions.json` | `system`, `era` (faction itself is the facet elsewhere) | `color` is a CSS variable name (e.g. `--f-ry`) picked from the palette already defined in `index.html` |
| `locations.json` | `faction`, `system`, `era` | `subtype` is `"core-world"` or `"frontier"` — controls which section it renders in |
| `cosmology.json` | `era` | `spoiler: true` adds the spoiler tag |
| `updates.json` | — | Rendered newest-first by `date` (ISO `YYYY-MM-DD`) |
| `fieldnotes.json` | — | Simple numbered list, no cross-linking |
| `links.json` | — | External further-reading cards |

Ids in `factions.json`, `locations.json`, and `cosmology.json` share one namespace — that's what lets a location's `related` array point at a faction, or a cosmology term point at a location.

## Project history

This started as a single hand-authored HTML file (kept in [`legacy/`](legacy/) for reference, along with the build notes that first diagnosed the "not really encyclopedic yet" problem). The current version follows that diagnosis: data + render, real cross-links, faceted filters synced to the URL, addressable entries.

## Research discipline

Facts here are cross-checked against multiple sources (Bethesda's own site, the UESP-run Starfield Wiki, and others) rather than pulled from memory alone — the Starfield fan wikis disagree with each other more often than you'd expect, so treat any entry that looks off as worth double-checking before you quote it.
