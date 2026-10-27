# Feature Design Pipeline — Playbook

The human-readable version of the `feature-design-pipeline` skill. This is the reusable stack: take a vague feature ask → engineer-ready design docs, grounded in reality, before any code. Worked example throughout: the RentOk **multi-party agreement** session that produced this.

---

## The shape of it

```
Ask → [1] Frame → [2] Ground code → [3] Research → [4] Brief → [5] Spec + diagrams → [6] Pre-mortem → [7] Lock → [7½] Review → [8] Persist → (handoff)
        brainstorming    graphify+understand  web-search subagents  brief   domain-modeling+api-design  pre-mortem   AskUser+ADR  doc-handoff  memory+ADR+mem0+repo
        &/or grill-with-docs (understand-domain) (competitive-analysis      + mermaid workflow/userflow  +identify-    →ADRs        -review    →planning-and-
                                                 if selection)             /sequence/state diagrams     assumptions                          task-breakdown
```

*(Grounded in the raw session transcripts: `brainstorming`, `pre-mortem`, `brief`, `doc-handoff-review`, `documentation-and-adrs`, `api-and-interface-design`, `planning-and-task-breakdown`, `ponytail`, and inline **mermaid** diagrams are what were actually used across the linked sessions. **GSD was never invoked** — don't assume it. grill / competitive-analysis / prd-architect are named as options.)*

Two ideas make it work, more than any single skill:
1. **Ground before you design.** Never write a spec against a guess. Map the real code and verify the real vendor/legal facts first — in subagents, so the raw bytes never clog your own window.
2. **Let reality reverse your decisions.** Lock decisions with the user as you go, but when a code map or a doc contradicts a locked call, *stop and surface it*. This is the highest-leverage move in the whole pipeline.

---

## Phase 0 — Route & gate
Invoke the process skills first (brainstorming before creative work). Hold one gate the entire time: **no code, schema, or migration until the docs exist and the user approves the direction.** Everything downstream trusts this gate.

## Phase 1 — Frame the ask (`brainstorming` and/or `grill-with-docs`)
Both are in the stack; they serve the same purpose (sharpen the ask) and pair well:
- **`brainstorming`** — the proven default (used every linked session). Open exploration: restate the ask in your own words until you both agree, one question at a time, recommended-first.
- **`grill-with-docs`** — when you're starting *from* an existing rough ask, ticket, or notes, this interrogates that document and updates domain terms/ADRs inline. **`grill-me`** for a pure blank slate.

Often the strongest move is **both**: grill the ticket to pressure-test it, then brainstorm the options it leaves open. `domain-modeling` fires the moment a new domain word shows up (don't defer it to the spec).

Split the ask into its real sub-changes — the multi-party session's "one feature" was actually three (N-party data model, per-party signing orchestration, where parties come from). Naming the sub-changes early keeps scope honest.

## Phase 2 — Ground the codebase (parallel subagents + graphify)
Spawn read-only subagents to map the *current* system: the entities, the flow, the integration, the exact constraint your change will hit. Demand `file:line` citations, forbid solutions, and ask each agent for **the biggest blocker** explicitly. Run independent maps in parallel.

Expect to be wrong once. This session's first map concluded "no e-sign exists"; the user corrected it; a re-map found the live Cashfree + in-house flows. **Grounding you can't trust is worse than none** — re-map the moment a fact is challenged.

Reach for the right grounding tool: **graphify** for structure (the RentOk graph already exists), **`/understand-domain`** when you need the business-flow a feature design almost always rests on, **`/understand`** for greenfield / no-graph codebases. Subagents run the queries so the raw bytes stay out of your window.

*Greenfield?* `/understand` + a landscape scan (prior art, comparable products, the standard data model).

## Phase 3 — Research externally (web-search subagents; competitive-analysis for selection)
Both kinds run before the spec asserts anything, each external claim with a source URL:
- **Verify a known vendor's contract** — targeted `web-search` + `ctx_fetch_and_index` subagents on the official docs (what actually worked here). Often your own code reveals the true contract faster than docs; use docs for the limits (max counts, webhook availability, per-item vs per-request). Docs can also *reverse* an assumption — a correctness-pushback moment (this session: "Cashfree multi-signer is sequential & all-Aadhaar", which killed the naive parallel-mixed design). Reach for `/competitive-analysis` or `/market-scan` when the question is *which* vendor to pick or the market landscape — not for confirming one you already use.
- **Domain/legal/market** — real use cases + compliance reality via targeted web-search. This adds scope you forgot (the `witness` role; two witnesses are standard in India) and cuts scope that's legally separate (e-stamping/registration are their own steps — out).

Agents say "not published / confirm with vendor" rather than guessing.

## Phase 4 — Brief (`brief`)
The WHY, in plain language a non-native engineer reads once and gets. Who's losing what today, the root cause, the bet, what you're *not* building, the traps (each cited to `file:line`). No engineering shape, no UI words — that's later. Keep it ~1 page.

## Phase 5 — Design spec (`domain-modeling` + `api-and-interface-design` + `ponytail`)
The HOW: the data model and migration, the API, the state/flow (with mermaid diagrams), edge cases, testing. Apply ponytail hard — smallest model that works, reuse what exists, no abstraction for a single implementation *unless the user explicitly asked for extensibility* (then build the seam, not the future methods). Mark every schema change for the user to approve before anyone migrates. When migrating live data, design it **additive-first with a dual-read window** so nothing in production breaks.

**Draw the flows.** A signing/state/approval flow is far clearer as a picture than prose — author **workflow, userflow, sequence, and state diagrams** as **inline mermaid** in the spec (what was used across the sessions: `sequenceDiagram`, `stateDiagram`, `flowchart`). For richer standalone boards, use `excalidraw-diagram`, `json-canvas`, or FigJam. Multi-surface features often warrant a dedicated **userflow doc** alongside the spec.

*Heavier doc needed?* If the feature is multi-surface or stakeholder-facing, the spec becomes a full PRD via **`/prd-architect` + `/rentok-prd` — always paired, never prd-architect alone on RentOk.** Light features stay at brief + spec + a couple of diagrams.

## Phase 6 — Pre-mortem (`pre-mortem`, + `identify-assumptions` for ambitious ones)
Imagine it shipped and failed; work backward. Tigers (real, launch-blocking) / paper tigers (overblown) / elephants (unspoken). Give each launch-blocking tiger an owner and a mitigation. Mandatory whenever money, legal, compliance, or a risky migration is in play. For ambitious features, add **`identify-assumptions` + `prioritize-assumptions`** — pre-mortem finds what kills it; identify-assumptions surfaces the risky *beliefs* the design rests on (a distinct, upstream lens). The pre-mortem often feeds fixes back into the spec.

## Phase 7 — Lock decisions (`AskUserQuestion` + ADRs)
At every fork, present options **recommended-first** with a one-line why, and let the user lock it. Record locked decisions in the spec as you go — and for the *architectural* ones (lasting blast radius), write an **ADR** via `documentation-and-adrs` so future engineers get the why, not just the what. And again: when later evidence reverses a lock, re-open it with the new facts — don't quietly build around a call made on wrong information.

## Phase 7½ — Adversarial doc review (`doc-handoff-review`)
Before the team reads the brief + spec, run `doc-handoff-review`: it fact-checks every claim against its cited source (zero-hallucination pass) and audits readability. This was actually used in the linked sessions — it's what catches a citation that drifted from its source or a passage no one can read once. Especially important when a doc is derived from another (a spec built off a contract, a brief off research), because the citations must re-verify against the parent.

## Phase 8 — Persist (memory + ADRs + mem0 + repo)
Never lose the work. Checkpoint, each place carrying the same locked decisions, verified facts, open questions, and next steps:
1. **File memory** — `project_<name>.md` + a `MEMORY.md` pointer. *(File-memory + ADRs are the documented expectation.)*
2. **ADRs** — architectural decisions recorded via `documentation-and-adrs`.
3. **mem0** — `add_memory`, stored verbatim (`infer: false`).
4. **Git repo** — docs pushed (private for internal work) with a README that has a **"resume here tomorrow"** section.

Then hand over a copy-paste resume prompt. Do this at any heavy-context boundary too, not just the end — protect quality by handing a fresh window a clean brief instead of pushing a degraded one.

## Handoff — plan, then build (never design → code)
Design docs done → an **approved plan** before any code — what was actually used here: **`planning-and-task-breakdown`** (spec → ordered tasks) or `writing-plans`, and **`/feature-dev`** for complex multi-file builds. *(GSD `/gsd:plan-phase` exists for spec-driven projects but wasn't used in these sessions — reach for it only if you're genuinely running a GSD project, don't assume it.)*

---

## Standing constraints (the whole time)
- **Subagents keep bytes out of your window** (`dispatching-parallel-agents` + `context-engineering`) — map and research in agents, not inline. Parallel + background for long jobs.
- **Ponytail / YAGNI** — smallest thing that works; defer ambitious scope explicitly instead of building it.
- **Correctness-pushback** (`doubt-driven-development`) — when evidence contradicts a locked decision, re-open it; don't design around a wrong call.
- **Plain-language docs** — brief = WHY (no code), spec = HOW (code ok); don't mix altitudes; no AI slop.
- **Recommended-first questions** — never a bare menu.
- **Verify before "done"** (`verification-before-completion`) + **review before push** — sanity-check what's going out; private for internal.

## The backbone: agent-skills suite
The build-shape skills come from **Addy Osmani's `agent-skills` plugin** (25 skills, called `agent-skills:<name>`): `api-and-interface-design`, `documentation-and-adrs`, `planning-and-task-breakdown`, `interview-me`, `idea-refine`, `source-driven-development`, `spec-driven-development`, `doubt-driven-development`. The process skills come from other plugins: `superpowers` (`brainstorming`, `dispatching-parallel-agents`, `writing-plans`, `verification-before-completion`), `ponytail`, `context-mode`, and `understand-anything` (`understand`, `understand-domain`). The framing skills `grilling`, `grill-with-docs` and `domain-modeling` come from Matt Pocock's skills; we ask one question at a time on top of `grilling`. Prefer the suite's dedicated skill over hand-rolling the behavior. The suite's build-phase members (`test-driven-development`, `incremental-implementation`, `shipping-and-launch`, `ci-cd`, `debugging`, `performance`, `observability`) are the *downstream* handoff, not this pipeline.

## Scaling
- **Small feature:** Phase 1 + a short brief + a lean spec. Skip the pre-mortem if there's no money/legal/migration risk.
- **Large feature (like multi-party):** the full set + flow diagrams + a plan (`planning-and-task-breakdown` / `writing-plans`) + a `doc-handoff-review` pass before the team reads it.

## What "done" looks like
A brief, a design spec **with its flows drawn out as diagrams**, a pre-mortem, a memory checkpoint (file + mem0), and a repo with a resume README — every decision traceable to a user lock and every external claim to a source.
