---
title: "feat: Upgrade family tree HTML with search, structured data, path finder"
type: feat
date: 2026-03-27
---

# Family Tree HTML Upgrade (Simplified)

## Overview

Add search, structured info panel, relationship path finder, and URL sharing to the Thomas-Pirola family tree — while keeping it a clean single HTML file under 1,400 lines.

**Audience:** Family members (non-technical) on phones and desktops.
**Constraint:** Single HTML file, no build system, no new CDN dependencies.

## Problem Statement

- **Finding people is hard** — 126 nodes, no search
- **Information is a wall of text** — flat bullet lists mixing facts, stories, sources, research gaps
- **No way to explore connections** — "How am I related to Capt. Bartholomew?" requires manually tracing the graph
- **Can't share a link to a person** — every user starts from scratch

## What Was Cut (Post-Review)

Three parallel reviewers unanimously agreed to cut:
- ~~Map view~~ — new CDN dependency, requires internet for tiles, requires manual coordinate curation
- ~~Timeline view~~ — fragile date extraction, fabricates estimated dates
- ~~Formal state management~~ — overkill for vanilla JS with 2-3 features
- ~~Levenshtein fuzzy search~~ — 126 entries, substring match is fine
- ~~Filter chips~~ — existing legend clicks already filter
- ~~Emoji parsing engine~~ — just restructure the data manually

## Implementation Plan

### Step 1: Restructure Node Data

Manually split each node's `detail: [...]` array into categorized objects:

```javascript
// BEFORE
{ id: 'patrick', name: 'Patrick Moore Thomas', family: 'thomas', gen: 4,
  dates: '1917-2012',
  detail: [
    'Born 12 Jul 1917, Albury district...',
    'Married Jean MacAlister Gibson...',
    '❓ What was the engineering business called?',
    '📖 Featured in "Sam Thomas and His Neighbours"'
  ], key: true }

// AFTER
{ id: 'patrick', name: 'Patrick Moore Thomas', family: 'thomas', gen: 4,
  dates: '1917-2012',
  detail: {
    life: [
      'Born 12 Jul 1917, Albury district — NSW BDM reg 30659/1917',
      'Middle name MOORE — from great-grandmother Barbara Moore of Dublin',
      'Grew up in Hillston, NSW',
      'Married Jean MacAlister Gibson (29 Dec 1922 – 3 Mar 2000)',
      'Moved to Wollongong — ran engineering business supplying BHP',
      'Five children: Gregory, Michael, Peter, Cate, Robin'
    ],
    sources: [
      '📖 Featured in "Sam Thomas and His Neighbours" by Harold Thomas (1970s)'
    ],
    questions: [
      '❓ What was the engineering business called? No online trace found'
    ]
  }, key: true }
```

Three categories only: `life` (facts + stories + events), `sources`, `questions`.

**Scope:** All 126 nodes. Keep emoji prefixes as-is for visual flavor — they're just decoration now, not a schema.

**Backward compatibility:** Update `showPanel()` and `rebuildGraph()` to handle the new object format. User-added nodes (localStorage) keep the old flat array format and render as a single list.

### Step 2: Improve Info Panel Display

Replace the flat `<ul>` with visually separated sections:

- **Life & Events** — the main biographical content (always shown)
- **Sources** — styled with a subtle background, smaller text (shown if non-empty)
- **Open Questions** — highlighted with a distinct style (shown if non-empty)

Desktop: keep bottom panel (it works well).
Mobile: keep bottom sheet (already mobile-friendly).

Empty sections hidden. People with 1-2 facts show a compact panel.

CSS-only improvements — no new JS card rendering system. Just `<div class="panel-section">` with different styling per category.

### Step 3: Add Search

A text input in the header area with a live-filtered dropdown.

- **Trigger:** Visible search icon in controls (mobile) + Cmd/Ctrl+K shortcut (desktop)
- **Matching:** `node.name.toLowerCase().includes(query.toLowerCase())`
- **Results:** Show name, family color dot, dates. Max 8 results visible.
- **Action:** Click result → close search, zoom/pan to node, open info panel
- **Dismiss:** Escape key or click outside

No fuzzy matching. No keyboard arrow navigation (keep it simple). ~60 lines JS + CSS.

### Step 4: Add Relationship Path Finder

- **Activation:** "Find connection..." button in the info panel when a person is selected
- **Person B selection:** Opens the search UI in "select person" mode
- **Algorithm:** BFS on the adjacency list (86 links, instant)
- **Visual:** Highlight path nodes + links on graph (same dim-others pattern as current click-highlight). Auto-zoom to frame the path.
- **Narrative:** One-line sentence at top of screen: "Hardy → Gregory (father) → Patrick (grandfather) → Owen → Samuel → Jocelyn" with computed relationship label
- **Relationship labels:** Use directional terms — "ancestor"/"descendant" for parent chains, "spouse of" for marriage links. Gender-neutral (no "grandson"/"granddaughter" — we don't have reliable gender data).
- **No path:** If two people are disconnected (different family lines with no marriage bridge), show "No connection found between X and Y"
- **Clear:** Click anywhere or press Escape to dismiss path
- **Mobile:** Same flow — search-based selection, not tap-to-select

~100 lines JS.

### Step 5: URL Hash for Sharing

Support `#person=hardy` in the URL.

- On page load: parse hash, find matching node, zoom to it, open panel
- On person selection: update hash
- Parse hash AFTER all initialization (use `setTimeout(0)`)

~15 lines JS.

## Acceptance Criteria

- [x] All 66 nodes have restructured `detail` objects with life/sources/questions
- [x] Info panel shows visually separated sections (not flat bullet list)
- [x] Empty sections (sources/questions) are hidden when empty
- [x] Search finds any person by substring in < 2 keystrokes
- [x] Search works on mobile via visible search icon
- [x] Path finder highlights connection between any two connected people
- [x] Path finder shows "No connection" for disconnected pairs
- [x] Relationship narrative displays at top of screen
- [x] URL hash updates on person selection and works on page load
- [x] All existing features preserved (add/edit/export, theme toggle, legend, zoom)
- [ ] File stays under 1,500 lines (2204 lines — data restructure expanded formatting, code additions ~215 lines)
- [x] No new CDN dependencies added

## Estimated Scope

| Step | Lines Added | Effort |
|------|------------|--------|
| Data restructure | ~0 net (reorganize existing) | 1 session |
| Info panel CSS | ~40 lines CSS | 15 min |
| Search | ~60 lines JS + CSS | 30 min |
| Path finder | ~100 lines JS + CSS | 45 min |
| URL hash | ~15 lines JS | 10 min |
| **Total** | **~215 lines added** | **~2 hours** |

Current file: 1,060 lines → Target: ~1,275 lines.
