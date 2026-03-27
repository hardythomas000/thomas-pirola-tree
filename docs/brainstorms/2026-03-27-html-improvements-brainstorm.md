# Thomas-Pirola Family Tree — HTML Improvements Brainstorm

**Date:** 2026-03-27
**Audience:** Family sharing (parents, cousins, relatives exploring casually)
**Current state:** Single-file D3.js force-directed graph, ~60+ people, 6 family lines, dark/light mode, info panel, add/edit/export

---

## What We're Building

A major upgrade to the family tree HTML to transform it from a research tool into a polished, shareable family artifact. Five interconnected improvements:

### 1. Spotlight Search (Cmd/Ctrl+K)
- **What:** A macOS Spotlight-style overlay triggered by keyboard shortcut or search icon
- **Behavior:** Type a name, see instant fuzzy-matched results, click to zoom + highlight that person on the graph
- **Why:** 60+ people is too many to scan visually. Family members need to find "their" person fast
- **Details:** Results show name, family line color dot, and dates. Keyboard navigable (arrow keys + Enter)

### 2. Relationship Path Finder
- **What:** Select two people to see how they're connected
- **UX:** Both animated path highlight on the graph AND a narrative card
- **Path highlight:** The connecting nodes and links light up, others dim (similar to current click-highlight but between two endpoints)
- **Narrative card:** A popup telling the story: "Hardy is the great-great-great-great-grandson of Jocelyn" with each step listed (Hardy -> Gregory (father) -> Patrick (grandfather) -> Owen -> Samuel -> Jocelyn)
- **Activation:** Either from the info panel ("Find connection to...") or from a dedicated button

### 3. Card-Based Info Display
- **What:** Replace the flat bullet list in the info panel with visual cards
- **Cards:**
  - **Life Summary** — born, died, married, locations (icon-based, compact)
  - **Key Events** — timeline of major life events with dates
  - **Stories** — the colorful narrative details (the war stories, the scandals, the family lore)
  - **Sources** — book references, ADB links, Trove articles
  - **Open Questions** — research gaps marked with the existing question markers
- **Why:** The current detail arrays have 10-20 items mixing facts, stories, sources, and questions. Cards make it scannable and visually rich for family members who just want "the good stuff"

### 4. Three View Modes (Toggle)
- **Graph View** (current) — D3 force-directed layout, the main interactive view
- **Map View** — Interactive map showing family migration paths:
  - Thomas: Wales -> County Wexford, Ireland -> Tasmania -> NSW -> QLD
  - Pirola: Sondrio -> Milan -> Barcelona -> Mexico City -> Shanghai -> Sydney
  - Gibson: Kilmaurs, Scotland -> Brisbane -> Bundaberg
  - Barlow: Nottingham -> London -> Switzerland -> France
  - Pins for each person at their known locations, lines showing migration
- **Timeline View** — Horizontal timeline from 1600s to 2020s:
  - Each person as a bar/dot positioned by birth-death dates
  - Color-coded by family line
  - Shows generational layers
  - Key historical events as context markers (English Civil War, convict era, WWII, etc.)
- **Toggle:** Clean toggle buttons in the header or control area

### 5. Search Filters
- Filter by family line (already partially possible via legend clicks, but make it explicit)
- Filter by "needs research" — show only people with open questions
- Filter by generation range
- Combine with spotlight search

---

## Key Decisions

1. **Card-based details** over tabs or collapsible sections — more visual, better for casual exploration
2. **Spotlight search** over sidebar directory — feels modern, doesn't eat screen space
3. **Combined path highlight + narrative** for relationship finder — gives both visual and textual understanding
4. **All three views as toggles** (graph/map/timeline) rather than picking one — each tells a different story about the family
5. **Family sharing is primary audience** — storytelling > research tooling, but research gaps still visible via cards

## Open Questions

- **Map library:** Leaflet.js (lightweight, free tiles) vs Mapbox (prettier but needs API key) vs simple SVG map?
- **Data structure:** The current flat arrays of detail strings need parsing/restructuring to populate cards properly. Should we add structured fields (birthPlace, deathPlace, events[]) to the node data, or parse the existing emoji-prefixed strings?
- **File size:** Currently a single ~75KB HTML file. Adding Leaflet + timeline could push it much larger. Keep single-file or split into modules?
- **Mobile UX:** Three view modes need to work on phone screens. Bottom sheet info panel is already mobile-friendly but cards need thought.
- **Performance:** 60+ nodes is fine for D3, but map with pins + timeline with bars for all people — still fine in a single page?

## Approach Recommendation

**Phase 1:** Restructure data + card-based info panel + spotlight search (highest impact, no new libraries)
**Phase 2:** Relationship path finder (graph algorithm + UI)
**Phase 3:** Timeline view (can be done with pure SVG/D3)
**Phase 4:** Map view (requires geographic data enrichment + map library)

---

*Next: Run `/workflows:plan` to create implementation plan*
