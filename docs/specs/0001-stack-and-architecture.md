# 0001. Stack and Architecture

**Date**: 2026-09-26
**Status**: Accepted

## Summary

This decision records the full-stack TypeScript architecture for the Multi-City Travel Planner and Budget Estimator. The project is a full-stack monolith: a Vite and React 18 single-page app on the frontend, an Express and TypeScript REST API on the backend, MySQL as the primary database accessed via Sequelize ORM, JWT authentication stored in HTTP-only cookies, and Socket.IO for real-time collaborative editing. Everything runs locally for now. The stack is simple, proven, and sized for a small team shipping in weeks.

## Context

The product is a greenfield full-stack web app with four core capabilities: itinerary building, budget tracking, activity discovery, and real-time collaboration. The team is small, the goal is to ship a working MVP in weeks, and all development is local with no deployment target yet.

The main forces shaping the stack choice are: TypeScript across the full stack for type safety and developer confidence; a relational database because the domain (trips, cities, days, budget entries, collaborators) is inherently relational with well defined entities and relationships; real-time collaborative editing as a stated scope feature that needs a persistent connection, ruling out serverless; and a monolith because the team size and timeline make distributed services unnecessary overhead.

A wrong stack decision here cascades into every later feature, so it is worth deliberating once and recording clearly. The goal is boring, proven technology that the team can build, maintain, and debug without surprises.

## Options considered

### Option 1: Full-stack monolith (React + Node.js, chosen)

One repository with a Vite and React frontend and an Express API backend. Both in TypeScript. Shared types can be extracted to a common package inside the monorepo if needed.

**Pros**:
- Simplest to build, run locally, and debug
- No network boundary to manage between frontend and backend during development
- One deployment unit when the time comes

**Cons**:
- Harder to scale frontend and backend independently if traffic profiles diverge (not a concern at this stage)

### Option 2: Separate frontend and backend repos

Independent repos with a separate deployment pipeline each.

**Pros**:
- Clean separation of concerns from day one

**Cons**:
- More overhead for a two-person or solo team: two repos, two CI pipelines, CORS to configure from day one, shared types require publishing a package

### Option 3: Next.js meta-framework

Next.js App Router serving both UI and API routes from a single framework.

**Pros**:
- SSR and SSG built in; good for public trip pages if that deferred feature is eventually added

**Cons**:
- More complex mental model (server components, client components, route handlers all mixed)
- Socket.IO needs a custom server, which bypasses Next.js's built-in server and loses some of its deployment story
- Overkill for an auth-walled SPA with one public landing page

## Decision

**Chosen option**: Option 1: Full-stack monolith (Vite + React 18 frontend, Express + TypeScript backend)

The monolith is the right call for a small team shipping fast. One repo, one dev server per side, no distributed complexity.

## Rationale

The domain is relational (trips reference cities, cities reference days, days reference activities and budget entries, trips reference collaborators) so PostgreSQL or MySQL is clearly correct. MySQL was chosen by the engineer. The real-time requirement (collaborative editing) demands a persistent WebSocket connection, which rules out Vercel serverless functions and confirms the monolith on a persistent Node.js server. Socket.IO is the mature, well-supported library for this in Node.js. Sequelize was chosen by the engineer as the ORM; it is a long-standing, battle-tested option for MySQL with TypeScript support. JWT in HTTP-only cookies is the standard stateless auth approach for a REST API: no server-side session store, secure against XSS because JavaScript cannot read HTTP-only cookies. TanStack Query handles server state on the frontend cleanly and avoids the boilerplate and cache management bugs that plain `useState` and `useEffect` accumulate. CSS Modules gives scoped styles without a runtime cost, appropriate for a focused product UI. Structured logging with pino or winston gives useful output in development without committing to a hosted observability service before deployment decisions are made.

## Proposed stack

| Layer | Choice | Reason |
|---|---|---|
| Language | TypeScript (frontend and backend) | Type safety across the full stack; catches data model mismatches at compile time |
| Frontend framework | Vite + React 18 (SPA) | Fast dev server, minimal config, correct for an auth-walled SPA |
| Backend framework | Express + TypeScript | Minimal, well understood, large ecosystem, easy to layer middleware onto |
| Primary database | MySQL (local) | Engineer preference; relational, ACID, handles the trips and budget domain well |
| ORM | Sequelize | Engineer preference; mature MySQL ORM with TypeScript support |
| Auth mechanism | JWT in HTTP-only cookies | Stateless sessions, secure against XSS, no session store needed |
| Real-time layer | Socket.IO (WebSockets) | Persistent connection for collaborative editing; mature rooms and reconnect support |
| Frontend state | TanStack Query (React Query) | Server state caching, loading, and error handling without boilerplate |
| Styling | CSS Modules | Scoped component styles, no runtime cost |
| Observability | pino or winston (structured logging) | Structured logs in development; add a hosted tool when deployment comes |
| Hosting | Local only (no deployment target yet) | Ship in weeks locally; deployment decision deferred |

## Consequences

**Positive**:
- Every layer is proven and well-documented; the team can find answers quickly
- TypeScript end to end means shared types can be extracted and used in both the Express API and the React app, eliminating a class of runtime shape mismatch bugs
- Monolith + local database means no cloud accounts, no environment variables to manage, and fast iteration

**Negative / tradeoffs**:
- MySQL over PostgreSQL loses advanced JSON operators and some indexing features; unlikely to matter at MVP scale but worth noting
- Sequelize is a heavier ORM than Prisma or Drizzle; its TypeScript ergonomics are not as tight (manual model definitions, less inference), which means more boilerplate in the data layer
- Socket.IO adds a dependency and a connection management concern; it must be integrated into the Express server correctly to share auth middleware with HTTP routes
- CSS Modules requires a build step (Vite handles this) and does not provide a design token system; the design system feature will need to define tokens separately

**Neutral**:
- No deployment target means no CI/CD, no environment variable management, and no staging environment yet; these will need decisions when the product is ready to deploy
- Structured logging without a hosted tool means logs are local only; errors in production (when it arrives) will need a tool added at that point

**Scalability ceiling and growth path**:
- The monolith on a single Node.js process handles thousands of concurrent users comfortably. When traffic grows beyond a single machine, the frontend (static files) is separated behind a CDN first, then the API is scaled vertically before horizontal scaling is considered.
- Socket.IO on a single process supports thousands of concurrent WebSocket connections. Horizontal scaling (multiple Node.js instances behind a load balancer) requires adding a Redis adapter to Socket.IO so that rooms and events are shared across instances. This is a well-understood extension point that does not require a rewrite; it is a one-time configuration change.
- MySQL scales to tens of millions of rows without specialised knowledge. Read replicas handle read-heavy workloads; a connection pool (already provided by Sequelize) is the main lever. A migration to a different database engine is not anticipated at MVP scale.
- Caching (Redis), background job queues, and multi-region are explicitly not in scope for the current build. They are additions, not rewrites, and are added only when a measured bottleneck demands them.

## Follow-up

- [ ] Sequelize TypeScript model definitions are more verbose than Prisma; consider evaluating Prisma before the data model feature is built, since switching after migrations start is painful
- [ ] Socket.IO auth middleware: ensure the JWT verification middleware is shared between HTTP routes and Socket.IO connection handlers; design this in the authentication spec
- [ ] Deployment target and hosting decision deferred; add a scope row and run `/architect` when the team is ready to ship
- [ ] Structured logging library choice (pino vs winston) is a RECOMMEND item: use pino; it is faster, has lower overhead, and produces JSON logs by default without configuration; winston is heavier and its API has more historical baggage
