# RentOk profile (overlay for `rentok-backend`)

Apply this on top of the core pipeline when working in `/Users/eazypg/rentok-backend`. It's the turnkey version — the conventions, paths, and checkpoint recipe already decided.

## Phase 2 grounding — RentOk specifics

- **Graphify first, always.** Read `graphify-out/GRAPH_REPORT.md` / query the graph before any grep. Never read `graphify-out/*.json` directly into context. Only open specific files the graph points to.
- **Domain footguns to check every time:** the `new_property_structure` duality (legacy `tenant.room` string vs `tenant_room` junction); the Room-is-everything hierarchy (Property→Flat→Room→Bed all in `Room` via `parent_id` + `unit_type`); tenant status enum (0 evicted, 1 active, 2 booking, 3 lead, 4 joining-req, 8 interested); amounts in **paise (integer)**; `TypeORM synchronize=false` (migrations are explicit).
- **Where things live:** routes `src/routes/`, controllers `src/controllers/`, services `src/services/` (business logic + DB), entities `src/entities/`, migrations `src/migrations/`, shared helpers `src/utils/commonFunctions`.

## Conventions the spec must honor

- **API shape:** `res.success(message, status, data)` / `res.error(...)` — defined in `src/utils/commonFunctions.ts`. Routes → validation middleware → controller (no business logic) → service.
- **DB:** no `SELECT *`; parameterized queries; new migration files (never edit old); index large-table queries; batch bulk ops; avoid N+1. **Flag every schema change to the user before migrating** — never auto-apply.
- **Payments:** amounts in paise; atomic transactions across tables; verify gateway status via API, not webhook payload alone; flag any change to Cashfree/Easebuzz/PayU/RazorpayX explicitly.
- **TypeScript:** strict, no `any`, explicit return types on exports, `??` not `||` for defaults, `unknown` in catch.
- **Team the docs are written for:** Jatin (Sr. backend), Parishi/Vivek/Harsh (fullstack), Shiv/Ansh (mobile), Devendra (QA), Sanchay (PM), Nimit (CTO). Write for readability by this team; prefer explicit over clever.

## Doc locations (match the existing feature families)

- Brief → `docs/<feature>-brief.md`
- Design spec → `docs/superpowers/specs/YYYY-MM-DD-<feature>-design.md`
- Pre-mortem → `docs/PreMortem-<feature>-<date>.md`
- Keep new docs alongside the related feature family (e.g. the `agreement-*` docs cluster in `docs/`).

## Mandatory chains (from global CLAUDE.md — non-negotiable on RentOk)

- **Frame the ask** → `brainstorming` is the proven default (used every session). Use `/grill-with-docs` when starting from a rough ticket/notes to interrogate, `/grill-me` for a blank slate. Same purpose; pick by whether there's a doc to grill.
- **Research before spec** → verify a *known* vendor with targeted `web-search`/doc-fetch subagents (what worked); `/competitive-analysis` only for vendor *selection* / landscape.
- **Pre-mortem after** any financial / compliance / launch spec (used every session).
- **Doc-handoff-review** on brief + spec before the team reads them (fact-check + readability).
- **Plan before execute** → `planning-and-task-breakdown` (spec → ordered tasks — what was used) or `writing-plans`; approve the plan before code. (`/gsd:*` exists but was not used in these sessions — don't assume it.)
- **Draw the flows** → workflow / userflow / sequence / state diagrams as inline **mermaid** in the spec; `excalidraw-diagram` / `json-canvas` / FigJam for richer standalone boards.
- **RentOk PRD** (only when a full PRD is warranted) → `/prd-architect` + `/rentok-prd` **together**, never prd-architect alone.
- **Graphify before grepping 5+ files;** `/understand-domain` for business-flow grounding.
- **Architectural decisions → ADRs** (`documentation-and-adrs`), not just memory (used in the linked sessions).

## Phase 8 checkpoint recipe (RentOk)

1. **File memory:** write `~/.claude/projects/-Users-eazypg-rentok-backend/memory/project_<feature>.md` (type: project — objective, locked decisions, verified facts w/ file:line, open questions, next steps, links via `[[other-memory]]`). Add a one-line pointer to that dir's `MEMORY.md`.
2. **mem0:** `mcp__mem0__add_memory` with `infer: false`, `metadata: {type: project, project: <feature>}` — same content, verbatim.
3. **Git:** internal RentOk doc-projects go **private** under the **`eazyapp-tech`** org (same home as the backend). `gh repo create eazyapp-tech/<name> --private --source=. --remote=origin --push` from a clean dir *outside* the backend tree (so it doesn't tangle with the backend git). README with a "resume here tomorrow" section.
4. Hand over a copy-paste resume prompt.

## Review gate

Run the code-reviewer agent before any push/PR (global standing rule). For docs, run `doc-handoff-review` (adversarial fact-check + readability) on the brief + spec before team handoff.
