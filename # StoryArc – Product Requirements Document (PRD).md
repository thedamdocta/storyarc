\# StoryArc – Product Requirements Document (PRD)

\#\# 0\. Meta  
\- \*\*Product Name:\*\* StoryArc  
\- \*\*Version:\*\* 1.0 (MVP Planning)  
\- \*\*Owner:\*\* (your name / user id)  
\- \*\*Last Updated:\*\* YYYY-MM-DD  
\- \*\*Status:\*\* Approved for Architecture & Story Cycle

\#\# 1\. Vision  
StoryArc empowers storytellers (screenwriters, novelists, game designers, attorneys, educators) to \*\*visually architect narratives\*\* through a modular, drag‑and‑drop timeline \+ kanban system with hierarchical structure (Storyline → Arc → Scene → Beat), rich metadata, and structural templates (Story Circle, 3‑Act, etc.). It provides clarity of plot progression, subplot interplay, character involvement, and structural integrity—acting like a “Figma for narrative structure”.

\#\# 2\. Goals & Objectives (MVP)  
| Goal | Objective | Success Metric (Initial) |  
|------|-----------|--------------------------|  
| Visual Narrative Design | Intuitive timeline (H & V orientation) | Create & reorder ≥50 scenes fluidly without perceptible lag (\<150ms frame updates) |  
| Modular Structure | CRUD hierarchy Storyline→Arc→Scene→Beat | 100% CRUD operations persisted; reorder with drag |  
| Structural Guidance | Apply template (e.g. Story Circle) | Template auto‑creates labeled sections in \<3s |  
| Organizational Flexibility | Kanban view (arc columns) | Dragging scene between arcs reflects in timeline instantly |  
| Rich Context | Metadata popups linking characters/locations/objects | Open/edit modal round trip \<500ms |  
| Entity Cross‑Linking | Character/location/prop usage tracking | Filter by character shows only related scenes |  
| Real-time Readiness | Supabase realtime foundation | Changes in another tab appear \<2s |  
| Fast Solo Build | Leverage Shadcn \+ Supabase | MVP feature set implemented within target sprint sequence |  
| Extensibility | Clear path to collaboration & AI assist | Architecture section lists concrete extension points |

\#\# 3\. Scope

\#\#\# 3.1 In Scope (MVP)  
\- Authentication (Supabase Auth) – email/password basic.  
\- Project (Storyline) management.  
\- Hierarchy: Arcs, Scenes, Beats (optional beats editing).  
\- Timeline view (horizontal first; vertical toggle).  
\- Kanban view (columns \= Arcs).  
\- Template application (Story Circle, base 3‑Act).  
\- Metadata modal (Scene & Beat) \+ entity linking.  
\- Entity libraries: Characters, Locations, Objects (props/evidence).  
\- Realtime subscriptions for optimistic multi-tab sync.  
\- Row-Level Security (RLS) for user isolation.  
\- Basic filtering (by character).  
\- Basic export (JSON outline).

\#\#\# 3.2 Future / Post-MVP (Not in v1 Implementation)  
\- Advanced branching / non-linear “what-if” timelines.  
\- AI narrative suggestions / pacing analysis.  
\- Script formatting / full screenplay editor.  
\- Collaboration (multi-user invites \+ roles).  
\- Advanced export (Fountain, Final Draft, PDF deck).  
\- Story analytics (tension curves, POV charts).  
\- Version branching & compare.  
\- Edge Functions for template batch insertion (optional later).  
\- Integrations (Final Draft, Notion sync, etc.).  
\- Media asset upload (images, boards) beyond basic.

\#\# 4\. User Personas & Key Use Cases

\#\#\# 4.1 Personas  
| Persona | Needs | Pain Points Today |  
|---------|-------|-------------------|  
| Screenwriter | Manage plot & subplots cohesively | Juggling separate docs/apps (Scrivener \+ Plottr) |  
| Novelist | Track character arcs over long narratives | Timeline fragmentation |  
| Game Narrative Designer | Parallel quest threads & dependencies | Hard to visualize concurrency |  
| Attorney | Fact chronology & witness/event linkage | Spreadsheets & static docs |  
| Educator/Coach | Teach structure frameworks | Students rely on static templates |

\#\#\# 4.2 Core Use Cases (Happy Path Summaries)  
1\. \*\*Create Story using Story Circle Template:\*\* Select template → arcs auto-seed → fill placeholder scenes.  
2\. \*\*Reorder Scenes Across Arcs:\*\* Drag on timeline; arc lane updates; kanban reflects move.  
3\. \*\*Link Characters to Scenes:\*\* Open scene modal → select characters → character filter highlights related scenes.  
4\. \*\*Switch Views:\*\* Toggle Timeline ↔ Kanban ↔ Orientation.  
5\. \*\*Sync Across Tabs:\*\* Open second tab → modify a scene in tab A → see update propagate.  
6\. \*\*Add Beats Inside Scene:\*\* Add micro beats; reorder; persist & reflect beat count on card.

\#\# 5\. Functional Requirements

\#\#\# 5.1 Hierarchy & Structure  
| ID | Requirement | Acceptance Criteria |  
|----|-------------|---------------------|  
| F-1 | Create Storyline | Storyline appears in list; auto owner; DB row created |  
| F-2 | CRUD Arcs | Create, rename, delete (cascade scenes), reorder via drag |  
| F-3 | CRUD Scenes | Create within arc; reorder; move between arcs w/ drag |  
| F-4 | CRUD Beats | Inline or modal add/edit/delete; order maintained |  
| F-5 | Apply Template | Selecting template seeds arcs/placeholder scenes in \<3s |  
| F-6 | Orientation Toggle | Horizontal ↔ Vertical without reload; layout persists |  
| F-7 | Kanban View | Columns \= arcs; moving card updates arc & timeline |  
| F-8 | Outline/Tree (Phase 2 optional) | Collapsible tree shows hierarchy |

\#\#\# 5.2 Metadata & Entities  
| ID | Requirement | Acceptance Criteria |  
|----|-------------|---------------------|  
| M-1 | Scene Modal | Editable fields: title, description, characters, locations, objects |  
| M-2 | Character CRUD | Persist per story; list view; deletion prevented if linked? (soft warning) |  
| M-3 | Linking System | Linking characters/locations updates join tables |  
| M-4 | Character Filter | Activating filter hides unrelated scenes or dims them |  
| M-5 | Beat Indicators | Scene card shows beat count if \>0 |  
| M-6 | Custom Fields (JSON) | Metadata JSON stored & retrievable (future extension) |

\#\#\# 5.3 Realtime & Persistence  
| ID | Requirement | Acceptance Criteria |  
|----|-------------|---------------------|  
| R-1 | Realtime Insert | New scene inserted appears in second tab \<2s |  
| R-2 | Realtime Update | Title change syncs across tabs \<2s |  
| R-3 | Offline Grace (Basic) | Network loss warns user; operations queue or blocked gracefully |  
| R-4 | Auth Session | Auth persists refresh (Supabase session) |

\#\#\# 5.4 Security  
| ID | Requirement | Acceptance Criteria |  
|----|-------------|---------------------|  
| S-1 | RLS Isolation | User cannot query another’s story rows (tested via attempted direct ID fetch) |  
| S-2 | Ownership Propagation | user\_id present on arcs/scenes (or enforce via join policy) |  
| S-3 | Delete Safety | Cascade deletes beats; confirm destructive actions |

\#\#\# 5.5 Performance  
| ID | Requirement | Acceptance Criteria |  
|----|-------------|---------------------|  
| P-1 | Timeline Render | 100 scenes render initial load \<1.5s TTI |  
| P-2 | Drag Frame Rate | Dragging scene remains \>50fps typical modern laptop |  
| P-3 | Modal Open | Scene modal opens \<300ms after click |

\#\#\# 5.6 Export  
| ID | Requirement | Acceptance Criteria |  
|----|-------------|---------------------|  
| X-1 | JSON Export | Download JSON with hierarchy & entity links |  
| X-2 | Plain Outline Export (Optional) | Markdown outline of arcs → scenes (stretch) |

\#\# 6\. Non-Functional Requirements  
| Category | Requirement |  
|----------|------------|  
| UX | Intuitive drag affordances & clear drop targets |  
| Accessibility | Keyboard navigation for reordering (phase 2\) |  
| Reliability | Auto-save on each mutation |  
| Maintainability | TypeScript types for schema; modular components |  
| Extensibility | Data model supports future collaborators & AI suggestions |  
| Privacy | No PII besides account email; minimal logging |

\#\# 7\. Constraints & Assumptions  
\- Supabase quotas adequate for MVP.  
\- Single-user active editing (multi-user later).  
\- Scenes may not have real timestamps (logical order first).  
\- Beats are optional: not all scenes will have them.

\#\# 8\. Risks & Mitigations  
| Risk | Impact | Mitigation |  
|------|--------|-----------|  
| Drag complexity timeline vs kanban diverges | Inconsistent ordering | Single canonical order\_index source; both views mutate same indices |  
| Template application partial failure | Data inconsistency | Wrap multi-insert in client batch; rollback on error; future RPC |  
| Realtime volume scaling | UI jitter | Debounce state merges; consider virtualization if \>300 visible items |  
| RLS misconfig | Data leak | Automated QA test script for cross-user access attempts |

\#\# 9\. Epics (Initial Definition)  
1\. \*\*Epic 1 – Core Data & Auth:\*\* Schema, auth integration, CRUD forms.  
2\. \*\*Epic 2 – Timeline (Horizontal) Core:\*\* Render arcs & scenes, drag reorder.  
3\. \*\*Epic 3 – Kanban View:\*\* Arc columns, drag scenes across arcs.  
4\. \*\*Epic 4 – Metadata & Entities:\*\* Modals, characters/locations/objects linking.  
5\. \*\*Epic 5 – Templates (Story Circle \+ 3-Act):\*\* Apply \+ placeholder generation.  
6\. \*\*Epic 6 – Realtime Sync:\*\* Subscriptions \+ multi-tab tests.  
7\. \*\*Epic 7 – Vertical Orientation & Filters:\*\* Orientation toggle & character filter.  
8\. \*\*Epic 8 – Export & Polish:\*\* JSON export, performance pass, confirmations.

\#\# 10\. Acceptance Criteria Summary (Per Epic)  
\- \*Epic 1:\* All tables \+ basic CRUD through temporary UI; RLS enforced.  
\- \*Epic 2:\* Drag reorder persists; moves reflected after refresh.  
\- \*Epic 3:\* Moving a scene via Kanban updates timeline position & arc\_id.  
\- \*Epic 4:\* Modal save updates linked lists; filter highlights scenes \<1s.  
\- \*Epic 5:\* Template seeds 8 Story Circle labeled placeholders.  
\- \*Epic 6:\* Two tabs show updates w/out manual refresh.  
\- \*Epic 7:\* Vertical mode retains ordering; character filter dims unrelated scenes.  
\- \*Epic 8:\* Export file matches current DB snapshot; main interactions performant.

\#\# 11\. Out of Scope (MVP reiteration)  
See Section 3.2.

\#\# 12\. Glossary  
\- \*\*Arc:\*\* Subplot or structural segment inside Storyline.  
\- \*\*Beat:\*\* Micro-event. Optional granular unit.  
\- \*\*Template:\*\* Structural blueprint that seeds arcs/scenes.

\#\# 13\. Future Enhancements (Backlog Seeds)  
\- Branching “what-if” timelines  
\- AI structural gap analysis  
\- Multi-user commenting / presence cursors  
\- Story analytics dashboards

\---  
\*End PRD\*

