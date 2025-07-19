### **`docs/architecture.md`**

markdown  
CopyEdit  
`# StoryArc – Architecture Document`

`## 0. Meta`  
`- **Version:** 1.0 (Aligned with PRD v1.0)`  
`- **Owner:** (your name)`  
`- **Last Updated:** YYYY-MM-DD`

`## 1. Architectural Overview`  
`Single Page Application (React + Shadcn UI) ←→ Supabase (Postgres + Auth + Realtime + Storage).`  
`No custom backend server initially; client uses Supabase JS SDK for CRUD & subscriptions. Future server logic via Supabase Edge Functions if needed.`

\[Browser React SPA\]  
 ├─ Auth (Supabase)  
 ├─ Data Fetch (REST / JS SDK)  
 ├─ Realtime Subscriptions (WebSocket)  
 └─ State Store (Zustand/Context)  
 ↕  
 \[Supabase Services\]  
 ├─ Postgres (Schema: stories, arcs, scenes, beats, entities)  
 ├─ RLS Policies  
 ├─ Realtime replication  
 └─ (Future) Edge Functions

sql  
CopyEdit

`## 2. Key Architectural Principles`  
`| Principle | Implementation |`  
`|----------|----------------|`  
``| **Single Source of Truth** | `order_index` in DB drives ordering across all views |``  
`| **Schema-Driven UI** | Types generated from DB (supabase type gen) feed TS interfaces |`  
`| **Optimistic + Realtime** | Immediate optimistic state change, reconcile via subscription |`  
`| **Security by Default** | RLS enforced on every table; client never bypasses |`  
`| **Extensibility** | Separate entity tables; no polymorphic over‑generalization early |`  
`| **Performance Simplicity** | Avoid premature virtualization until threshold reached |`

`## 3. Tech Stack`  
`| Layer | Choice | Rationale |`  
`|-------|--------|-----------|`  
`| Framework | React + Vite + TS | Fast dev; type safety; Replit-friendly |`  
`| UI | Shadcn UI (Radix + Tailwind) | Accessible primitives + rapid composition |`  
`| State | Zustand (or React Context) | Lightweight store, simple selectors |`  
`| Auth/DB/Realtime | Supabase | Managed Postgres + RLS + websockets |`  
``| Drag & Drop | `@dnd-kit` (preferred) | Modern, composable, sensor API |``  
`| Forms | React Hook Form + Zod (optional) | Declarative validation |`  
`| Testing (later) | Vitest/Jest + RTL | Component & util coverage |`  
`| Build/Deploy | Vite build → Vercel/Netlify | CDN delivery, quick deploy |`  
`| Export | Plain JS (JSON), optional Markdown generator | No external deps needed MVP |`

`## 4. Data Model (Schema)`

`### 4.1 Tables`

`**stories**`  
`| Field | Type | Notes |`  
`|-------|------|-------|`  
`| id | uuid PK | |`  
`| user_id | uuid FK -> auth.users | Owner / RLS |`  
`| title | text | |`  
`| description | text nullable | |`  
`| created_at | timestamptz default now() | |`

`**arcs**`  
`| Field | Type | Notes |`  
`|-------|------|-------|`  
`| id | uuid PK |`  
`| story_id | uuid FK stories(id) ON DELETE CASCADE |`  
`| user_id | uuid (redundant) |`  
`| title | text |`  
`| order_index | int |`  
`| color | text nullable |`  
`| created_at | timestamptz |`

`**scenes**`  
`| Field | Type | Notes |`  
`|-------|------|-------|`  
`| id | uuid PK |`  
`| arc_id | uuid FK arcs(id) ON DELETE CASCADE |`  
`| story_id | uuid FK stories(id) (denormalized) |`  
`| user_id | uuid |`  
`| title | text |`  
`| description | text |`  
`| order_index | int |`  
`| timestamp | timestamptz nullable (for real chronology) |`  
`| metadata_json | jsonb nullable |`  
`| created_at | timestamptz |`

`**beats**`  
`| Field | Type | Notes |`  
`|-------|------|-------|`  
`| id | uuid PK |`  
`| scene_id | uuid FK scenes(id) ON DELETE CASCADE |`  
`| story_id | uuid |`  
`| user_id | uuid |`  
`| summary | text |`  
`| order_index | int |`  
`| created_at | timestamptz |`

`**characters / locations / objects** (same pattern)`  
`| Field | Type |`  
`|-------|------|`  
`| id | uuid PK |`  
`| story_id | uuid |`  
`| user_id | uuid |`  
`| name | text |`  
`| description | text |`  
`| created_at | timestamptz |`

`**scene_characters / scene_locations / scene_objects** (join tables)`  
`| Field | Type | Notes |`  
`|-------|------|-------|`  
`| scene_id | uuid FK scenes |`  
`| character_id/location_id/object_id | uuid FK respective |`  
`| PRIMARY KEY(scene_id, character_id) | composite |`

`### 4.2 Indexing`  
`` - `arcs (story_id, order_index)` ``  
`` - `scenes (arc_id, order_index)` ``  
`` - `beats (scene_id, order_index)` ``  
``- `scene_characters (character_id)` for reverse lookup.``  
``- Add partial index if filtering by character frequently: `scenes(story_id)` is implicit; join should be fine early.``

`### 4.3 Ordering Strategy`  
``Simple integer `order_index`. On reordering:``  
`1. Local reorder array.`  
`2. Batch update changed rows.`  
`3. Long-term optimization: gap strategy (e.g. allocating 100 increments) or fractional: not required MVP.`

`## 5. RLS Policies (Conceptual)`  
`**Each table**:`  
```` ```sql ````  
`ALTER TABLE <table> ENABLE ROW LEVEL SECURITY;`

`CREATE POLICY "owner_all_<table>"`  
`ON <table>`  
`FOR ALL`  
`USING (user_id = auth.uid())`  
`WITH CHECK (user_id = auth.uid());`

**NOTE:** Ensure `user_id` denormalization on insert (client includes it). Optionally, DB trigger to enforce `user_id = (SELECT user_id FROM stories WHERE id = NEW.story_id)` for integrity.

## **6\. Application Layer Components**

| Component | Responsibility |
| ----- | ----- |
| `<AppRoot>` | Supabase client init; auth session gating |
| `<AuthGate>` | Show login / register (Supabase UI or custom) |
| `<StoryLoader>` | Fetch & subscribe story data; supply store |
| `<Toolbar>` | View toggles, template apply, export |
| `<TimelineView>` | Lanes (arcs) \+ draggable scene cards (horizontal) |
| `<TimelineLane>` | Layout scenes for one arc |
| `<SceneCard>` | Reusable scene display (timeline/kanban) |
| `<BeatList>` | Inline beat editing inside modal |
| `<KanbanView>` | Arc columns with draggable scenes |
| `<SceneModal>` | Edit scene metadata, beats, links |
| `<CharacterPanel>` | CRUD & filter toggle |
| `<EntityTagSelector>` | Multi-select link control |
| `<TemplateWizard>` | Apply structural template |
| `<ExportDialog>` | JSON/outline export |
| `<FilterBar>` | Character/location filters |
| `<OrientationToggle>` | Horizontal ↔ Vertical switch |
| `<RealtimeProvider>` | Hook subscription management |

### **6.1 State Shape (Zustand Example)**

ts  
CopyEdit  
`type StoryState = {`  
  `storyId: string;`  
  `arcs: Arc[];`  
  `scenes: Scene[];`  
  `beats: Beat[];`  
  `characters: Character[];`  
  `locations: Location[];`  
  `objects: StoryObject[];`  
  `filters: { characterIds: string[] };`  
  `setArcs(...);`  
  `setScenes(...);`  
  `applySceneUpdate(scene: ScenePartial);`  
  `// etc.`  
`};`

### **6.2 Realtime Hook**

ts  
CopyEdit  
`useEffect(() => {`  
  `const channel = supabase`  
    ``.channel(`story:${storyId}`)``  
    ``.on('postgres_changes', { event: '*', schema: 'public', table: 'scenes', filter: `story_id=eq.${storyId}` }, handleScene)``  
    ``.on('postgres_changes', { event: '*', schema: 'public', table: 'arcs', filter: `story_id=eq.${storyId}` }, handleArc)``  
    `// repeat for beats/entities`  
    `.subscribe();`  
  `return () => supabase.removeChannel(channel);`  
`}, [storyId]);`

## **7\. Template Application Flow (Story Circle)**

**Client Steps:**

1. User selects “Apply Story Circle”.

2. For each of 8 steps create arc rows (or scenes if arcs already represent acts).

3. Optionally seed placeholder scenes (one per step).

4. If failure in mid-seed: rollback by deleting inserted arc IDs gathered.

Future: Wrap in Postgres function with transaction.

## **8\. Drag & Drop Architecture**

* **Kanban:** `@dnd-kit` sortable context per column; on drag end compute new `arc_id` & order indices.

* **Timeline:** Use `@dnd-kit` with custom horizontal sensors; drop target \= lane bounds; compute insertion index by pointer x relative to lane width (divide by card width / gap).

* **Optimistic Update:** Update local ordering immediately; fire batch update; reconcile with incoming realtime (idempotent).

## **9\. Filtering & Highlighting**

Filtering by character:

* Compute set of scene\_ids from `scene_characters` join.

* If filter active: non-matching scenes get `opacity-40 pointer-events-none` (Tailwind).

* Performance: Precompute map `{characterId: Set<sceneId>}` on data load.

## **10\. Export Strategy**

**JSON:**

json  
CopyEdit  
`{`  
  `"story": {...},`  
  `"arcs": [...],`  
  `"scenes": [`  
    `{"id":"...", "arc_id":"...", "beats":[...], "characters":[...], "locations":[...], "objects":[...]}`  
  `]`  
`}`

**Outline (Markdown)** (future):

shell  
CopyEdit  
`# Story Title`  
`## Arc: Main Plot`  
`### Scene 1 - Title`  
`- Beat: ...`

## **11\. Performance Considerations**

| Area | Strategy |
| ----- | ----- |
| Rendering Many Scenes | Keyed list \+ minimal re-renders (Zustand selectors) |
| Drag Performance | Translate3d transforms (CSS), avoid heavy box shadows |
| Data Fetch | Parallel fetch arcs/scenes/entities; load beats lazily on modal open (optional) |
| Large Data Future | Virtualization (react-virtualized) threshold \>200 visible cards |

## **12\. Error Handling**

| Type | Handling |
| ----- | ----- |
| Network Failure | Toast \+ disable mutating buttons; retry queue (simple) |
| Auth Expiry | Supabase auto refresh; if fail redirect to login |
| Insert Failure (Template) | Collect created IDs; delete rollback |
| Realtime Desync | Periodic (manual trigger) “Refresh Data” action |

## **13\. Logging & Debug**

* Dev mode: console debug enabled (flag in config).

* Optionally store last 20 actions in an in-memory log panel for troubleshooting.

## **14\. Security Notes**

* Enforce HTTPS in deployment.

* No secrets in client except anon key.

* For future AI features: move secret API keys to Edge Functions.

## **15\. Extensibility Points**

| Future Feature | Hook |
| ----- | ----- |
| Collaboration | Add `story_collaborators` \+ extend RLS |
| AI Suggestions | Add `suggestions` table & Edge Function call |
| Versioning | `story_versions` snapshot table or event sourcing (scenes\_history) |
| Advanced Templates | `templates` & `template_steps` tables; generic seeding routine |
| Analytics | Derived materialized views (e.g. character scene count) |

## **16\. Testing Strategy (High-Level)**

| Layer | Tests |
| ----- | ----- |
| Schema | Verify constraints (unique, FK cascade) |
| RLS | Automated queries as different users (QA script) |
| Store | Reducer/state mutation unit tests |
| Components | SceneCard drag, modal open/close |
| Realtime | Two simulated clients (JSDOM) updating scene title |
| E2E (Later) | Cypress: create story → add arc → add scene → reorder → export |

## **17\. Initial Implementation Order (Epics → Stories)**

1. **Epic 1 (Core Data & Auth):** schema creation script, Supabase config, basic CRUD forms.

2. **Epic 2 (Timeline Core):** read arcs/scenes, render horizontal, drag reorder, persist order.

3. **Epic 3 (Kanban):** columns from arcs, drag across arcs.

4. **Epic 4 (Metadata & Entities):** modals, entity CRUD, linking tables.

5. **Epic 5 (Templates):** Story Circle seeding.

6. **Epic 6 (Realtime):** subscriptions \+ test multi-tab.

7. **Epic 7 (Orientation & Filters):** vertical layout \+ character filter.

8. **Epic 8 (Export & Polish):** JSON export, confirmation dialogs, perf tuning.

## **18\. Open Questions / Decisions to Finalize**

| Question | Decision Needed |
| ----- | ----- |
| Beat inline editing vs modal only? | (Decide before Epic 4\) |
| Denormalize user\_id on all child tables? | (Recommend YES for simpler RLS) |
| Use order\_index gaps? | Start simple; revisit if reorder thrash emerges |
| Scenes timeline spacing metric? | Order index → constant width (MVP), later scalable axis |

## **19\. Appendix: Sample Type Definitions**

ts  
CopyEdit  
`type Story = { id: string; user_id: string; title: string; description?: string };`  
`type Arc = { id: string; story_id: string; user_id: string; title: string; order_index: number; color?: string };`  
`type Scene = { id: string; arc_id: string; story_id: string; user_id: string; title: string; description: string; order_index: number; metadata_json?: any };`  
`type Beat = { id: string; scene_id: string; story_id: string; user_id: string; summary: string; order_index: number };`  
`type Character = { id: string; story_id: string; user_id: string; name: string; description?: string };`

## **20\. Change Log**

| Date | Change | Author |
| ----- | ----- | ----- |
| YYYY-MM-DD | Initial architecture doc extracted & separated from PRD | You |

