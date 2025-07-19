# StoryArc

> **A modular narrative architecture workspace.** Visual timeline + kanban + structural templates (Story Circle, 3‑Act, etc.) for Storyline → Arc → Scene → Beat. Built with React, Shadcn UI, Supabase, and the BMad agent workflow.

---

## Table of Contents

1. [Overview](#overview)
2. [Core Value Proposition](#core-value-proposition)
3. [Key Features (MVP)](#key-features-mvp)
4. [Future Enhancements](#future-enhancements)
5. [Architecture Summary](#architecture-summary)
6. [Tech Stack](#tech-stack)
7. [Directory Structure](#directory-structure)
8. [Getting Started](#getting-started)
9. [Environment Variables](#environment-variables)
10. [Data Model (Concise)](#data-model-concise)
11. [Development Workflow (BMad Method)](#development-workflow-bmad-method)
12. [Epics Roadmap](#epics-roadmap)
13. [Scripts & Tooling](#scripts--tooling)
14. [Coding Standards (Summary)](#coding-standards-summary)
15. [Contributing](#contributing)
16. [Security & RLS](#security--rls)
17. [Export & Extensibility](#export--extensibility)
18. [FAQ](#faq)
19. [License](#license)

---

## Overview

StoryArc is a **visual story architecture workspace** that lets writers, game designers, attorneys, and educators craft narratives using a *single, unified model*. Instead of juggling spreadsheets, cards, and documents, users manipulate a **timeline** (horizontal or vertical) and a **kanban** board sharing the same underlying data. Structural templates (e.g. Dan Harmon’s Story Circle) seed arcs and placeholder scenes, while rich metadata popups connect characters, locations, objects, and beats.

---

## Core Value Proposition

| Pain                                | Traditional Tool Gap                                       | StoryArc Solution                                     |
| ----------------------------------- | ---------------------------------------------------------- | ----------------------------------------------------- |
| Fragmented outlining vs structure   | Multiple disjoint tools (timeline app + doc + index cards) | Single canonical hierarchy (Storyline→Arc→Scene→Beat) |
| Hard to visualize subplot interplay | Linear script editors only                                 | Multi-lane timeline (arcs as lanes) + drag & reorder  |
| Structural guidance missing         | Blank-page syndrome                                        | Built‑in structure templates (Story Circle, 3‑Act)    |
| Character consistency drift         | Manual tracking                                            | Linked entities & filtering highlights coverage       |
| Reordering friction                 | Renumbering scenes manually                                | Drag-and-drop + realtime persistence                  |

---

## Key Features (MVP)

* **Hierarchy:** Storyline → Arc → Scene → Beat (optional granular beats).
* **Timeline View:** Horizontal (default) + vertical toggle; arcs as swimlanes.
* **Kanban View:** Columns = Arcs; drag scenes between arcs.
* **Structural Templates:** Auto-seed Story Circle, 3‑Act (extensible templates).
* **Metadata Popups:** Scene modal with description, beats list, linked characters/locations/objects.
* **Entity Libraries:** Characters, Locations, Objects with cross-linking.
* **Realtime Sync:** Supabase Postgres changes → WebSocket updates (multi-tab readiness).
* **RLS Security:** Row-Level Security ensures isolation per user.
* **JSON Export:** Download full story hierarchy snapshot.
* **Extensibility Hooks:** Designed for AI assistance, multi-user collaboration, analytics.

---

## Future Enhancements

| Category             | Examples                                                      |
| -------------------- | ------------------------------------------------------------- |
| Collaboration        | Shared stories, presence cursors, comments                    |
| AI Assistance        | Beat suggestions, structural gap analysis, pacing diagnostics |
| Advanced Exports     | Fountain / Final Draft / PDF outline decks                    |
| Branching Narratives | Parallel / conditional timeline branches & compare diff views |
| Analytics            | Character screen-time metrics, tension curves, POV shifts     |
| Versioning           | Story snapshots & diff; branching “what-if” scenarios         |
| Rich Media           | Storyboard panels, image uploads via Supabase storage         |

---

## Architecture Summary

```
[ React SPA ] ── Supabase JS SDK ──► [ Supabase Postgres ]
      │              ▲                (Auth, RLS, Realtime)
      │              │
      │  WebSocket Realtime (scenes/arcs/beats changes)
      ▼
 State Store (Zustand/Context) ← Drag + DnD Kit Events
```

**Principles:** Single source of truth (DB order\_index), optimistic updates + realtime reconciliation, schema-driven TypeScript types, security by default (RLS everywhere), modular UI components.

---

## Tech Stack

| Layer                 | Tool                                | Notes                                     |
| --------------------- | ----------------------------------- | ----------------------------------------- |
| Framework             | React + TypeScript + Vite           | Fast dev, HMR, type safety                |
| UI System             | Shadcn UI (Radix + Tailwind)        | Accessible primitives, design consistency |
| State                 | Zustand (or Context)                | Lightweight, fine-grained selectors       |
| Drag & Drop           | @dnd-kit                            | Flexible sensors; timeline & kanban reuse |
| Backend               | Supabase (Postgres, Auth, Realtime) | Zero-maintenance BaaS                     |
| Validation (optional) | Zod + React Hook Form               | Strongly typed forms                      |
| Testing (later)       | Vitest/Jest + RTL                   | Unit + component tests                    |
| Deployment            | Vercel / Netlify (static)           | SPA build output                          |

---

## Directory Structure

```
root/
  docs/
    prd.md
    architecture.md
    technical-preferences.md
    prd/                # sharded PRD sections (auto)
    architecture/       # sharded Architecture sections (auto)
    epics/              # epic-n-title.md files
    stories/            # story files generated by SM
  src/
    components/
      timeline/
      kanban/
      modals/
      entities/
    hooks/
    state/
    lib/
    pages/ (or routes/ if using a router)
  public/
  supabase/ (SQL migrations, seed scripts, RLS policies)
  .env.local
  package.json
```

---

## Getting Started

### 1. Clone & Install

```bash
pnpm install  # or npm install / yarn
```

### 2. Setup Supabase

1. Create Supabase project.
2. Copy **Project URL** & **Anon Key** into `.env.local` (see below).
3. Run SQL migration (schema + RLS) via Supabase SQL editor or CLI.

### 3. Configure .env

Create `.env.local` with variables (see [Environment Variables](#environment-variables)).

### 4. Run Dev Server

```bash
pnpm dev
```

Open `http://localhost:5173` (Vite default) or Replit-provided URL.

### 5. Auth

Register a user (email/password). RLS uses `auth.uid()`; all story rows must contain your `user_id`.

### 6. Apply Template

Create a Storyline → Click **Templates** → Choose *Story Circle* → arcs + placeholders seed.

### 7. Begin Outlining

Add scenes, drag to reorder, open scene modal for metadata, create beats, link characters.

---

## Environment Variables

| Variable                    | Purpose                  |
| --------------------------- | ------------------------ |
| `VITE_SUPABASE_URL`         | Supabase Project URL     |
| `VITE_SUPABASE_ANON_KEY`    | Supabase anon public key |
| `VITE_APP_NAME` (optional)  | Branding / window title  |
| `VITE_LOG_LEVEL` (optional) | debug / info             |

*Never commit service\_role or private keys.*

---

## Data Model (Concise)

**stories → arcs → scenes → beats** plus entity tables (characters, locations, objects) and join tables (`scene_characters`, etc.). All child rows denormalize `user_id` & `story_id` for simpler RLS.

Key columns: `order_index` drives ordering; `metadata_json` supports extension; join tables provide many‑to‑many linking.

---

## Development Workflow (BMad Method)

1. **Planning Phase:** PRD + Architecture (DONE) → shard into `docs/prd/` & `docs/architecture/`.
2. **Epic Definition:** Create `docs/epics/epic-n-*.md` (ordered story groups).
3. **Story Cycle:** For each epic: `SM` generates Story (Draft) → Approve → Dev implements → QA reviews → Mark Done → Next Story.
4. **Context Discipline:** One active Story file at a time keeps AI context narrow and precise.
5. **Artifacts:** Story files contain: Context, Acceptance Criteria, Tasks, Dev Notes, Testing Plan, QA Results.

**Automation:** Use `npx bmad-method install` to keep agent bundles updated. Keep PRD & Architecture authoritative; adjust them *before* generating new stories if scope changes.

---

## Epics Roadmap

| # | Epic                       | Outcome                                         |
| - | -------------------------- | ----------------------------------------------- |
| 1 | Core Data & Auth           | Schema + CRUD + Auth UI                         |
| 2 | Timeline Core (Horizontal) | Render lanes + reorder scenes                   |
| 3 | Kanban View                | Arc columns + cross-arc drag                    |
| 4 | Metadata & Entities        | Modals + entity linking + filters groundwork    |
| 5 | Templates                  | Story Circle & 3‑Act seeding                    |
| 6 | Realtime Sync              | Multi-tab updates, optimistic reconcile         |
| 7 | Orientation & Filters      | Vertical layout + character filter highlighting |
| 8 | Export & Polish            | JSON export, confirmations, perf pass           |

---

## Scripts & Tooling

| Script        | Command          | Description               |
| ------------- | ---------------- | ------------------------- |
| Dev           | `pnpm dev`       | Run Vite dev server       |
| Build         | `pnpm build`     | Production build          |
| Preview       | `pnpm preview`   | Preview production bundle |
| Lint          | `pnpm lint`      | ESLint (configure rules)  |
| Type Check    | `pnpm typecheck` | `tsc --noEmit`            |
| Test (future) | `pnpm test`      | Unit/component tests      |

---

## Coding Standards (Summary)

* **Language:** TypeScript strict.
* **Naming:** `PascalCase` components, `camelCase` functions/vars, `snake_case` DB columns.
* **Files:** One component per file; co-locate styles; avoid deep nesting >3 levels.
* **Imports:** Absolute path aliases (e.g. `@/components/...`).
* **State:** No direct mutation; functions named `setX`, `updateX`.
* **UI:** Use Shadcn primitives first; custom styles with Tailwind utilities; maintain accessible labels.
* **Error Handling:** Graceful toasts + console for dev.

(Full version in `docs/architecture/coding-standards.md`).

---

## Contributing

1. Fork & clone.
2. Create feature branch: `feat/timeline-drag`.
3. Run/update migrations if schema changes.
4. Write / update story file (if using BMad) before coding.
5. Submit PR with: description, linked story ID, screenshots/gifs.
6. QA checklist: ✅ Passing tests, ✅ No console errors, ✅ RLS unaffected.

*For solo dev: still follow branch discipline to keep history clean.*

---

## Security & RLS

* **Row-Level Security:** All tables `ENABLE RLS`; policies restrict CRUD to `user_id = auth.uid()`.
* **Denormalization:** `user_id` on arcs/scenes/beats simplifies policy predicates.
* **Auth Persistence:** Supabase auto-refresh; logout clears local session.
* **Future Collaboration:** Add `story_collaborators` join + extend policy with OR condition.

---

## Export & Extensibility

* **JSON Export:** Traverses in-memory state → prompts download.
* **Future:** Markdown outline, Fountain script export, analytics materialized views, AI suggestions via Edge Functions.
* **Integration Hooks:** Add template definitions, AI suggestion records, or collaborator roles without schema upheaval (core tables remain stable).

---

## FAQ

**Q:** Why not store beats as nested JSON?
**A:** Normalized beats enable ordering, filtering, realtime, and future analytics.

**Q:** Why `order_index` vs timestamps?
**A:** Many stories are logical sequences (not absolute time); index is deterministic & easily rebalanced.

**Q:** How is drag reorder persisted?
**A:** Optimistic local reorder → batch update changed `order_index` rows → realtime confirms (idempotent).

**Q:** Can I branch alternate versions?
**A:** Planned: future `story_versions` snapshot table (see roadmap).

---

## License

MIT © YEAR YOUR\_NAME

---

### Quick Start TL;DR

```bash
npx bmad-method install   # (optional AI workflow tooling)
pnpm install
echo "VITE_SUPABASE_URL=..." >> .env.local
echo "VITE_SUPABASE_ANON_KEY=..." >> .env.local
pnpm dev  # open browser, create story, apply template
```

---

**Happy outlining!**
