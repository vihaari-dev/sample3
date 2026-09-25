# Scope: Multi-City Travel Planner & Budget Estimator

A responsive web app that lets individual travelers, families, and groups plan multi-city trips with an itinerary builder and live budget tracker, replacing the mess of spreadsheets and booking tabs.

**Build approach:** Tracer Bullet (each feature is a thin but fully real vertical slice through every layer, proving integration early).
**Workflow:** Beta (after `/develop`, run `/check verify` then `/test`). The project default level of rigor. `/architect` is the recommended first stop for any feature with a real decision, but skippable when you already know the build. Any feature can carry its own tag (e.g. `· GA`) to do more or less.

_These are recommendations to keep your build orderly, not requirements. Skip anything that does not fit: if you already know how to build a feature, use `/develop` and skip `/architect`. You decide when a feature is `done`._

## At a glance

| # | Feature | Phase | Status |
|---|---------|-------|--------|
| 1 | Stack and architecture | Foundation | planned |
| 2 | Coding standards and tooling | Foundation | planned |
| 3 | Data model | Foundation | planned |
| 4 | Design system and UI foundation | Foundation | planned |
| 5 | Authentication | Slice 1 | planned |
| 6 | Trip creation and itinerary builder | Slice 1 | planned |
| 7 | Budget tracking | Slice 1 | planned |
| 8 | Public landing page | Slice 1 | planned |
| 9 | Activity discovery | Slice 2 | planned |
| 10 | Real-time collaboration and sharing | Slice 3 | planned |

## Foundations

### 1. Stack and architecture · needs a decision
Decide the full-stack TypeScript architecture (React frontend, Node.js/Express backend, database, real-time layer, hosting) and scaffold a runnable project so every later slice builds on real structure.
**Done when:** the stack is recorded in a spec and the empty scaffold boots locally, the dev server starts, and build passes.
- [ ] Decide the stack (spec): `/architect stack and architecture`

### 2. Coding standards and tooling
Capture conventions from the real scaffolded project, then install lint, format, type enforcement, and pre-commit hooks.
**Done when:** root `AGENTS.md` reflects the real stack and tooling, and lint, format, and pre-commit run clean on an empty project.
- [ ] Capture conventions and tooling choices: `/audit`
- [ ] Install the tooling: `/develop tooling`
- [ ] Check it runs clean: `/test`

### 3. Data model · needs a decision
Core entities every feature builds on: users, trips, cities, itinerary days, activities (curated), budget entries, collaborators.
**Done when:** entities and relationships support all four slices (auth, itinerary, budget, activities, collaboration) without a breaking migration.
- [ ] Design it (spec): `/architect data model`

### 4. Design system and UI foundation · needs a decision
Visual language, layout primitives, and base components (typography, color, spacing, forms, cards, navigation) so every screen feels cohesive and is accessible.
**Done when:** `design.md` covers type, color, spacing, and components; base components handle focus and keyboard navigation; renders cleanly on mobile and desktop.
- [ ] Design it (spec): `/architect design system and UI foundation`

## Slice 1: Core trip loop

### 5. Authentication · needs a decision
Email and password sign-up, sign-in, sign-out, and password reset. The gateway to every other feature.
**Done when:** a user can create an account, sign in, reset their password, and sign out; sessions persist across page refreshes; unauthenticated users are redirected to sign in.
- [ ] Design it (spec): `/architect authentication`

### 6. Trip creation and itinerary builder · needs a decision
Create a named multi-city trip, add destination cities with arrival and departure dates, reorder cities, and assign daily activities and notes to each day. The core of the product.
**Done when:** a signed-in user can create a trip, add and reorder cities with date ranges, add activities or notes to specific days, edit or delete any entry, and see the full itinerary on a timeline view; empty and error states render.
- [ ] Design it (spec): `/architect trip creation and itinerary builder`

### 7. Budget tracking · needs a decision
Attach estimated costs (flights, accommodation, activities, food, transport, other) to each city or trip day; see a live per-city and total trip cost breakdown.
**Done when:** a user can add, edit, and delete budget entries by category and city; totals update live; a clear breakdown by category and city is visible; the overall trip total is always shown.
- [ ] Design it (spec): `/architect budget tracking`

### 8. Public landing page
A public marketing page explaining what the product does and linking to sign up. No auth required. Lightweight SEO metadata.
**Done when:** the landing page renders with a headline, value proposition, and a call to action to sign up; page title, description meta tag, and Open Graph tags are set; it is accessible and responsive.
- [ ] Build it: `/develop public landing page`
- [ ] Verify it: `/check verify public landing page`
- [ ] Test it: `/test public landing page`

## Slice 2: Activity discovery

### 9. Activity discovery · needs a decision
Search the internal curated database of local attractions, restaurants, and activities by city or category and add results directly to a day on the itinerary.
**Done when:** a user can search activities by city and category, browse results with a name and description, add an activity to a specific itinerary day, and remove it; empty and no-results states render.
- [ ] Design it (spec): `/architect activity discovery`

## Slice 3: Collaboration and sharing

### 10. Real-time collaboration and sharing · needs a decision
Invite other users to co-edit a trip, see their changes live without refreshing, and generate a read-only share link for non-collaborators.
**Done when:** a trip owner can invite collaborators by email; all collaborators see edits in real time; a read-only share link lets anyone view the itinerary without an account; collaborators can be removed.
- [ ] Design it (spec): `/architect real-time collaboration and sharing`

## Deferred
Out of scope for the current build pass, kept so the plan stays honest.
- **Freemium billing and plans**: free tier vs paid with gated features · needs a decision · GA
- **Email notifications**: reminders before a trip, collaboration invites via email · needs a decision
- **Public trip pages**: shareable itineraries indexed by search engines · needs a decision
- **Mobile app**: native iOS and Android clients · needs a decision
- **Admin panel**: internal tooling to manage users and curated activity data · needs a decision
- **Product analytics**: track sign-ups, trips created, and shares · needs a decision

## Legend

**The decision box.** Every feature carries exactly one, the sub-task whose label ends with `(spec)`. Skills locate it by that `(spec)` suffix. Every other box is an execution box.

**Feature lifecycle**: the scope updates as a feature moves.

| State | Set by | The feature shows |
|---|---|---|
| `planned` · needs a decision | `/scope` | one box: `Design it (spec): /architect <feature>` |
| `in-progress` (designed) | `/architect` at spec capture | `Design it` ticked; spec linked; `Build it: /develop <feature>` + 2 to 5 milestones; `Verify it` and `Test it` boxes |
| `in-progress` (building) | `/develop` | milestone sub-boxes tick one by one; code pointer filled |
| `in-progress` (verified) | `/check verify` | `Build it` and milestones ticked; `Verify it` ticked |
| `done` | you, when you decide it is | boxes you ran ticked; skipped ones marked skipped |

- **Next step** = the first unticked box (always a command or a tracked milestone).
- **needs a decision** = run `/architect` first; otherwise straight to `/develop`.
- **Atomic build tasks live in the spec's `## Build plan`, not here**: the scope carries only the milestone rollup.
- **Status** `planned` → `in-progress` → `done`, plus `existing` (pre-workflow) and `dropped` (de-scoped, kept for history).
- **Workflow tier tag** (e.g. `· GA`) sets a feature's rigor above the project default; no tag inherits Beta.
- **Pointer line** (`spec <n> · code in <path>`): added by `/architect` and `/develop` when they run.

## References

### Practices and standards
- **Foundations before features**: stack, data model, and design system decided before any slice begins; a wrong data model is the most expensive thing to redo.
- **Tracer Bullet first slice as walking skeleton**: the thin real core loop (auth → create → budget) proves the full pipe (DB, API, UI) before adding breadth, retiring integration risk early.
- **Slice ordering by dependency**: auth gates everything; itinerary builder and budget are the stated MVP pair; activity discovery extends the builder; real-time collaboration requires a stable data and auth foundation, so it comes last.
- **Deferred structure for speed**: freemium billing, email notifications, and analytics kept out of the initial slices so the team ships itinerary plus budget in weeks without billing complexity.
- **Curated internal database over live API**: removes third-party API key management, rate limits, and cost risk from the MVP; can be swapped in a later slice once the product is proven.
