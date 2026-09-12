# Don't Panic! — Build Notes & Continuation Plan

For whoever (or whichever AI) picks this up in a code editor next. Written after two rounds of building it as a single HTML file inside a chat session — this is what's actually there, what's broken, and what I'd change about the architecture before pushing it further toward "encyclopedic."

## What this is

A pocket-guide-style reference to the *Starfield* universe (Bethesda's game), styled to echo the visual language of bethesda.net/game/starfield: dark navy body, cream nav strip, gold accent, condensed uppercase display type over a serif body face. Currently one self-contained `.html` file — no build step, no dependencies, opens straight in a browser.

## The bug that made search look broken

Found and fixed before handing this off, but worth understanding because it'll bite again if the same pattern gets reused: the search box has a decorative magnifying-glass `<svg>` icon positioned with `position:absolute` over the left edge of a plain `position:static` `<input>`.

CSS stacking rule that trips people up: **a positioned element (`absolute`/`relative`/`fixed`) always paints above a non-positioned sibling, regardless of source order**, unless you say otherwise. The `<input>` came *after* the `<svg>` in the markup, but because the `<svg>` was positioned and the `<input>` wasn't, the icon was actually sitting on top — and it had no `pointer-events: none`, so a click anywhere on that ~15px icon swallowed the click instead of focusing the input. Anyone who did the natural thing and clicked the magnifying glass to "activate" search got nothing.

Fix was one line: `pointer-events: none` on the `.search-box svg`. Already applied in the file you're getting. General rule for the rebuild: any purely decorative overlay (icons, badges, ambient background shapes) needs `pointer-events: none` unless it's meant to be clickable.

## Current structure (single file)

```
<div class="identbar">      thin top strip, brand mark only
<div class="toolbar">        sticky: section nav + search input + faction chips + result count
<header class="hero">        "DON'T PANIC" title, standing intro text
<section id="orientation">   setting/lore prose + 5 stat tiles
<section id="state-of-play"> 3 "patch note" style cards — current game updates/DLC
<section id="factions">      8 cards, one per faction
<section id="core-worlds">   4 detailed entries (the 4 main settled systems)
<section id="frontier">      11 compact cards (secondary systems/locations)
<section id="cosmology">     5 term/definition entries (endgame lore, spoiler-flagged)
<section id="field-notes">   8 short numbered tips
<section id="further-reading"> 6 outbound links to real Starfield resources
<footer>
<script>                     ~40 lines vanilla JS, no dependencies
```

**Filtering model:** every filterable block carries class `item`; faction-taggable ones also carry `data-faction="uc|freestar|crimson|ryujin|va-ruun|constellation|ecliptic|terran-armada|independent"`. The script does a flat show/hide pass on every `.item` against (a) the search box's text, matched against `el.textContent`, and (b) the currently active faction chip. No indexing, no debounce — fine at ~40 items, would not scale to hundreds.

## Content inventory so far

Roughly 34 curated entries total: 8 factions, 4 core-world writeups, 11 frontier locations, 5 cosmology terms, 8 field notes, 3 "state of play" update cards, 6 external links. That's a curated highlights reel — Starfield itself has 120 star systems and ~1,700 catalogued planets/moons, so "encyclopedic" coverage is a different order of magnitude from what's here now. Worth deciding early whether the goal is *deeper* (more detail per existing entry, real cross-referencing) or *wider* (systematically covering far more locations) — they pull the architecture in different directions.

Research for everything currently in the file was cross-checked against multiple sources (PCGamesN, Fextralife, StarfieldDB, Game8, Bethesda's own site, the UESP-run Starfield Wiki, GamesRadar, GameRevolution, PrimaGames, Xbox Wire, Dexerto) rather than pulled from memory alone — a few first-pass assumptions (which system Vectera's in, who actually runs HopeTown, which planet the Red Mile is on) turned out wrong on the first guess and got corrected against sources. Same discipline is worth keeping up if this expands — the Starfield fan wikis disagree with each other more often than you'd expect.

## Why it reads as "not really encyclopedic" yet

1. **No cross-linking.** "Neon" is mentioned in the Ryujin Industries card, the Volii core-world entry, and a field note, but none of those mentions link to each other. An encyclopedia's whole value is the web of cross-references; right now this is just a list of independent blurbs that happen to share proper nouns.
2. **Filtering is single-axis.** One row of faction chips. No way to filter by system, by location type (city / hostile outpost / neutral station / DLC-new), or by era (base game / Shattered Space / Terran Armada) at the same time.
3. **No entry has its own address.** Everything is a card in a scroll of cards — there's no `#neon` page or deep-linkable unit smaller than a whole section. Can't send someone a link straight to one entry.
4. **Flat data baked into markup.** Every fact lives hand-typed inside a `<div>`. Adding, restructuring, or generating cross-references means editing HTML by hand — that's the ceiling this format hits.

## Recommended architecture for the next pass

Move from "hardcoded cards" to **data + render**:

**1. A data file** (`data.json`, or split by type — `factions.json`, `locations.json`, `cosmology.json`) where each entry is an object, roughly:

```json
{
  "id": "neon",
  "type": "location",
  "name": "Neon",
  "system": "Volii",
  "planet": "Volii Alpha",
  "faction": "ryujin",
  "era": "base",
  "tags": ["core-world", "vice", "drugs", "city"],
  "summary": "One-line card blurb.",
  "body": "Full paragraph(s), can reference other ids in prose.",
  "related": ["ryujin-industries", "aurora-drug", "the-key"]
}
```

**2. A render pass** that turns that data into both the card grid *and* an addressable unit per entry (`id="neon"` at minimum; a proper detail view if this becomes a real site). Two ways to get cross-references without hand-linking every mention:
   - Explicit: use the `related` array to render a "See also" row per entry (simple, reliable, editorial control).
   - Automatic: build a `{name → id}` dictionary from all entry names, run it over each `body` string, and wrap the *first* occurrence of any other entry's name in `<a href="#id">`. More "encyclopedic," more moving parts — watch for false positives on short/common names.

**3. Real faceted filtering** — swap the single chip row for independent facets (checkbox groups or `<select multiple>`, one per facet: Faction, System, Type, Era), combined with AND logic across facets and OR within a facet. Sync the active filter state to the URL query string (`?faction=crimson&type=hostile-outpost`) so a filtered view is itself shareable — a genuinely useful "encyclopedic reference" feature that the current version doesn't have at all.

**4. Keep the visual design system** — the token set (colors, the Anton/Barlow/Source Serif 4/IBM Plex Mono type pairing) tested fine visually and matches what was asked for (mirroring bethesda.net's look). No need to redo that part; it can sit on top of whatever data/render layer replaces the hardcoded markup.

If this is going into an actual code editor/repo rather than staying a single file, a static site generator (11ty, Astro) or even a small vanilla build script that reads the JSON and stamps out HTML would remove the "hand-edit a 900-line file to add one planet" ceiling this version is already hitting.

## Other rough edges worth knowing about

- Only the Frontier section shows a "no matches" message when a filter empties it out; Factions/Core Worlds/Field Notes just go silently blank. Minor, but noticeable once you're actually using the filters.
- No debounce on the search input — a non-issue at 40 items, worth adding if the dataset grows into the hundreds.
- `<html lang="en" data-theme="dark">` is hard-pinned dark — deliberate (mirrors the Bethesda site, which doesn't have a light mode either), not an oversight, but flag it if a light mode ever gets requested.

## Files delivered alongside this note

- `dont-panic-starfield-guide.html` — current working version, search bug fixed, safe to use as the visual/content starting point.
