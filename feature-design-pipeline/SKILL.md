---
name: feature-design-pipeline
description: >-
  Use when turning a vague, ambitious, or high-stakes feature ask into engineer-ready,
  codebase-grounded design docs BEFORE any code — the full pipeline of brainstorm →
  codebase grounding → vendor/legal research → brief → design spec → pre-mortem → persist.
  Trigger whenever the user says "design this feature", "scope this", "think through my ask
  and state it back", "let's plan X before we build", "write the brief/spec/pre-mortem for X",
  or kicks off any non-trivial new feature, integration, or modification where the WHY and HOW
  aren't written down yet. Works on RentOk backend (auto-applies the RentOk profile) and on
  outside greenfield/brownfield projects. Do NOT use for quick one-file edits, isolated bug
  fixes, or when the work is already specced and only implementation remains.
---

# Feature Design Pipeline

Turn a rough ask into **engineer-ready design docs** — grounded in the actual codebase, the actual vendor APIs, and the actual domain/legal reality — *before a line of code is written*. This is the recipe distilled from a real end-to-end feature-scoping session (multi-party agreements): the sequence, the skills, the subagent patterns, and the rules that kept it honest.

This skill **orchestrates other skills**. It doesn't replace `brainstorming`, `brief`, or `pre-mortem` — it invokes them in the right order with the right grounding in between, and adds the connective tissue (parallel codebase maps, vendor/legal research, decision-locking, persistence) that those individual skills don't cover.

## What this is / isn't

- **Is:** the path from "I want feature X" to a brief + design spec (with flow diagrams) + pre-mortem an engineer can build from. Ends by handing off to a plan (`agent-skills:planning-and-task-breakdown` or `superpowers:writing-plans`) — it does not write production code.
- **Isn't:** a coding skill, a bug-fix flow, or a substitute for the build (plan mode, then `superpowers:executing-plans`). If implementation is already specced, skip this.

## The one hard gate

**No implementation, schema migration, or code scaffolding until the design docs exist and the user has approved the direction.** Every decision along the way is the user's to confirm — you propose, they lock. This is what makes the output trustworthy instead of confident fiction.

## The pipeline

Run these in order. Each phase feeds the next. Skip a phase only when it genuinely doesn't apply (noted per phase), and say so.

| # | Phase | Skill / tool | Why it's here |
|---|---|---|---|
| 0 | **Route & gate** | `using-superpowers`, `skill-router` | Invoke process skills before answering; hold the no-code gate. |
| 1 | **Frame the ask** | `brainstorming` + `AskUserQuestion` (proven default, used every session) **and/or `grilling` (with `domain-modeling` when ADRs and a glossary are wanted)** when a rough ask / ticket / notes exist to pressure-test (`grilling` for a pure blank slate) — both are in the stack | Restate the ask, one question at a time, recommended-first, until you agree. brainstorming for open exploration; grill-with-docs to interrogate an existing doc. Often do both — grill the ticket, then brainstorm options. `domain-modeling` fires the moment new domain terms surface. Split the ask into its real sub-changes. |
| 2 | **Ground the codebase** | parallel `Agent` subagents + `graphify` | Map the *current* system before designing. Never design against a guess. Re-map when a map turns out wrong. graphify = structure and communities; for the business flow, query the graph and read the callers along the money path. *(Greenfield → graphify on the empty repo is pointless; do a landscape scan instead.)* |
| 3 | **Research externally** | targeted `web-search` / `ctx_fetch_and_index` / Context7 subagents to verify the vendor's real contract + domain/legal facts (what actually worked here); **`/competitive-analysis` or `/market-scan`** when the question is landscape or vendor *selection* | **Research before spec** — every external claim gets a source, *before* the spec asserts it. For a *known* vendor, verify the contract from its docs + your own code; competitive-analysis is for choosing between options, not confirming one you already use. |
| 4 | **Brief** — always | `brief` | The WHY, plain language, operator-pain, codebase-cited traps. Used every design session; don't skip it. |
| 5 | **Design spec + flow diagrams** | `domain-modeling` + `api-and-interface-design` + `ponytail`; **workflow / userflow / sequence / state diagrams** — inline **mermaid** (what was used), or `excalidraw-diagram` / `json-canvas` / FigJam for richer standalone boards; full PRD? → `/prd-architect` + `/rentok-prd` (paired, never alone on RentOk) | The HOW: model, migration, API, and the flows drawn out as diagrams (a signing/state flow is far clearer as a picture). Light feature → brief + spec + a couple mermaid diagrams. Multi-surface → the PRD pair + userflow docs. |
| 6 | **Pre-mortem** | `pre-mortem` (+ **`identify-assumptions` / `prioritize-assumptions`** for ambitious features) | What kills it (pre-mortem, the hard gate) + the risky beliefs the design rests on (identify-assumptions, upstream lens). Mandatory when money, legal, compliance, or a risky migration is involved. |
| 7 | **Lock decisions** | `AskUserQuestion`; architectural locks → **`documentation-and-adrs`** (ADR) | Recommend + let the user lock. Record *architectural* decisions as ADRs, not just memory. When research reverses an earlier lock, **surface it — don't silently inherit the wrong call.** |
| 7½ | **Adversarial doc review** | **`doc-handoff-review`** | Before the team reads the brief + spec: fact-check every claim against its sources + a readability pass. Actually used in the linked sessions — catches hallucinated citations and dense passages. |
| 8 | **Persist** | file memory + ADRs (documented core); `mem0` + `gh` private repo + resume README (convention) | Checkpoint so a fresh session resumes instantly. File-memory + ADRs are the documented expectation; mem0 + repo are the belt-and-suspenders convention. |
| → | **Handoff** | **`agent-skills:planning-and-task-breakdown`** or **`superpowers:writing-plans`** (spec → ordered tasks), then `superpowers:executing-plans` or `subagent-driven-development` for the build | Design docs → **approved plan** → build. Never jump design → code. |

## Built on two suites: superpowers (process) and Addy Osmani's agent-skills (build shape)

Both are cherry-picked on purpose and both plugins are on in Claude Code; the same files are linked into Codex and Cursor. Sanchay's rule from 2026-07-18: for any brief or spec that becomes code, run the matching agent-skills skill **in parallel with** the PM skill, never after, or the spec comes out correct but build-naive.

- **Phase 1 (frame):** `superpowers:brainstorming` · `grilling` (one question at a time, a recommended answer each) · `agent-skills:interview-me` · `agent-skills:idea-refine` (divergent then convergent, surfaces assumptions)
- **Phase 2 (ground):** `graphify` · `superpowers:dispatching-parallel-agents` · `agent-skills:context-engineering` · `context-mode` for large reads
- **Phase 3 (research):** `agent-skills:source-driven-development` (every decision cites a source) · `defuddle` for pages · Context7 for library docs
- **Phase 5 (spec):** `brief` · `prd-architect` with `rentok-prd` · `agent-skills:spec-driven-development` · `agent-skills:api-and-interface-design` (API shape) · `agent-skills:frontend-ui-engineering` (offline, drafts, partial save, optimistic updates, skeletons) · `frontend-design` or `interface-design` for the visual layer
- **Phase 6 (risk):** `pre-mortem` · `doc-handoff-review` · `agent-skills:security-and-hardening` (anything touching money, auth, or tenant data) · `agent-skills:doubt-driven-development` (adversarial re-review)
- **Phase 7 (lock/persist):** `obsidian` (write the doc to the vault) · `agent-skills:documentation-and-adrs` · `agent-skills:planning-and-task-breakdown` or `superpowers:writing-plans` · `superpowers:verification-before-completion`
- **Downstream (build — NOT this pipeline):** `agent-skills:test-driven-development` or `superpowers:test-driven-development`, `agent-skills:incremental-implementation`, `superpowers:executing-plans`, `agent-skills:code-review-and-quality`, `agent-skills:shipping-and-launch`, `agent-skills:ci-cd-and-automation`, `agent-skills:debugging-and-error-recovery`, `agent-skills:performance-optimization`, `agent-skills:observability-and-instrumentation`; reviewers `agent-skills:code-reviewer`, `agent-skills:security-auditor`, `agent-skills:test-engineer`, `agent-skills:web-performance-auditor`.

## Profiles — pick based on where you are

The pipeline is the same; the grounding step (Phase 2) and the persistence step (Phase 8) differ.

- **Brownfield (existing codebase, incl. RentOk backend):** full Phase 2 — map the current system with parallel subagents before designing. In RentOk, read `references/rentok-profile.md` — it bakes in graphify-first, the `docs/` locations, team names, `res.success`/paise/migration conventions, and the exact memory + mem0 + `gh` checkpoint you should produce.
- **Greenfield (new project, no code yet):** Phase 2 becomes a **landscape scan** — prior art, comparable products, the shape of the domain — instead of a code map. Everything else holds. Persistence still applies (repo + memory).

Read `references/rentok-profile.md` when working in `rentok-backend`. Read `references/subagent-prompts.md` before spawning any Phase-2/3 subagent — it has battle-tested prompt templates you should adapt, not rewrite from scratch.

## Standing constraints (apply throughout, not a phase)

These ran the whole session and are what made it efficient and correct:

- **Context-mode discipline** (`context-engineering` + `dispatching-parallel-agents`). Do heavy reading/mapping/research in **subagents**, not in your own window. A subagent returns the conclusion; the raw file bytes and web pages stay out of your context. This is what lets the pipeline stay sharp across many large maps. Run independent maps/research **in parallel** (and use background agents for long web research).
- **Ponytail (YAGNI).** Design the smallest thing that works. Reuse what the codebase already has. Don't invent abstractions for one implementation — *unless the user explicitly asks for extensibility* (then build the seam, not the future). Flag deferrable scope instead of building it.
- **Plain-language docs.** Briefs say WHY in words a non-native, plain-vocab engineer reads once and gets. Specs carry the HOW. Don't mix altitudes. No AI slop.
- **AskUserQuestion, recommended-first.** Every decision point leads with a "(Recommended)" option and a one-line justification — never a bare menu.
- **Correctness-pushback** (`doubt-driven-development`). When a code map, vendor doc, or legal fact contradicts an earlier locked decision, stop and surface it with the new evidence + a fresh recommendation. Do not quietly design around a call the user made on wrong information. This is the single highest-value move in the whole pipeline — it happened twice this session and both times saved a wrong build.
- **Verify before you claim done** (`verification-before-completion`). Don't assert a doc is complete/accurate without checking; evidence before assertions.
- **Review before push.** Before any `gh` push / PR, sanity-check what's going out (private for anything internal).

## Persistence (Phase 8) — never lose the work

At the end (or at any heavy context boundary), checkpoint in three places so a fresh session resumes instantly:

1. **File memory** — a `project_<name>.md` in the project's memory dir + a one-line pointer in `MEMORY.md`. Locked decisions, verified facts, open questions, next steps. *(This + ADRs are the documented expectation.)*
2. **ADRs** — record each *architectural* decision (the ones with lasting blast radius) via `documentation-and-adrs`, so future engineers see the why, not just the what.
3. **mem0** (`mcp__mem0__add_memory`, `infer: false`) — the same checkpoint, stored verbatim, so it's queryable cross-session.
4. **Git** — the docs in a repo (private for internal work) with a **README that has a "resume here" section**: locked decisions, open decision, next steps, who-does-what.

Then hand the user a copy-paste resume prompt.

## Typical output set

A run usually produces: a **brief** (`*-brief.md`), a **design spec** (`specs/YYYY-MM-DD-*-design.md`) with **workflow / userflow / sequence / state diagrams** (mermaid inline, or standalone flow docs), a **pre-mortem** (`PreMortem-*.md`), a **memory checkpoint** (file + mem0), and a **repo** with a resume README. Scale down for small features (brief + short spec + a diagram or two), up for large ones (add the plan via `planning-and-task-breakdown`).

## Pointers

- `PLAYBOOK.md` — the human-readable, printable version of this pipeline with the reasoning per phase. Read or share this when you want the *why* laid out linearly.
- `references/subagent-prompts.md` — reusable prompt templates for the Phase-2 codebase maps and Phase-3 vendor/legal research. Adapt these; don't reinvent them.
- `references/rentok-profile.md` — the RentOk-backend overlay (conventions, paths, checkpoint recipe).
