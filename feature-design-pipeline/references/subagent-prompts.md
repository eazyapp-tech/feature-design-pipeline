# Reusable subagent prompts

These are the Phase-2 (codebase grounding) and Phase-3 (external research) prompts, generalized from a real session. Adapt the `<angle-bracket>` bits; keep the structure. The structure is the point: **demand file:line citations, forbid solutions, ask for the biggest constraint/mismatch explicitly, and tell the agent to say "I don't know" rather than guess.**

Spawn independent maps/research **in parallel** in one turn. Use `run_in_background: true` for long web research so you keep working. Prefer the `Explore` agent for codebase maps (read-only, fans out) and `general-purpose` for web research.

---

## Phase 2 — Codebase map (brownfield)

> I'm scoping `<feature>` for `<system>`. I need a precise map of the CURRENT state — do NOT propose solutions, map only.
>
> Use the knowledge graph / index first (`<graphify or equivalent>`), then read specific files. Be thorough across naming variants: `<list every likely synonym — e.g. agreement, esign, e-sign, signatory, party, firstParty>`.
>
> Report concisely, with **path:line** for each:
> 1. **`<primary entity>`** — which table(s)/models represent it, key columns, how many `<X>` it supports today and *how* (fixed columns? junction table? JSON?). Any status/role enums.
> 2. **`<the integration / flow in question>`** — which vendor/module, where the request is created, where the callback/webhook lands, how state is tracked.
> 3. **`<the config/UI the user named>`** — where it lives, which entity, which endpoint.
> 4. **`<related in-flight work>`** — read `<doc paths>`; summarize what's locked and relevant.
> 5. **Generation / delivery / routes** — the key files for create / send / callback / fetch-status.
>
> End with: **the biggest constraint** a `<the change>` design would hit (e.g. schema assumes exactly N, vendor supports one X per request, 1:1 relation blocks many). Cite path:line. If something is ambiguous, say so — don't guess.

**When a map comes back wrong** (the user corrects a fact): re-spawn immediately, name the correction, and tell the agent the prior conclusion was false. Grounding you can't trust is worse than no grounding.

---

> **Which tool:** to verify a *known* vendor's contract (the common case), use the targeted `web-search` + doc-fetch subagent below — that's what actually worked. Reach for `/competitive-analysis` or `/market-scan` only when the question is *which* vendor to choose or the market landscape, not confirming one you already use.

## Phase 3a — Vendor / API contract

> Research `<vendor + product>`'s API to confirm exact capabilities — we rely on this for `<feature>`. Use official docs (fetch them), prefer official over blogs. Answer precisely with a **source URL per claim**:
> 1. Does it support `<the thing we need — e.g. many signers / batch / webhooks>`? Exact fields, limits, max counts.
> 2. `<ordering / concurrency / per-item options>` — what's actually controllable?
> 3. `<auth / modes>` — per-item or per-request? Enumerate allowed values.
> 4. **Webhook vs polling** — does it push events? Names, payload, signature header. Or must we poll?
> 5. **Expiry / retry / rate limits / cost** — real numbers, or say "not published, confirm with vendor".
> 6. Status lifecycle — the full set of states.
>
> Where docs are ambiguous, say so — do NOT fabricate. Return a structured report.

**Key lesson:** the *code you already have* often reveals the true vendor contract (what you actually send/receive) faster than docs. Map the code first; use docs to fill gaps (max limits, webhook availability, whether an option is per-item). Docs can also *reverse* an assumption the code seemed to support — that's a correctness-pushback moment.

---

## Phase 3b — Domain / legal / market research

> Research practical `<jurisdiction>` use cases and `<legal/compliance>` norms for `<domain>`, to ground a product design. Use web search + fetch reputable/official sources. Answer with a **source per claim**:
> 1. Real-world scenarios and who/what is involved in each.
> 2. `<the compliance question — signatures, consent, retention, disclosures>`.
> 3. Legal validity of `<the mechanism we plan>` — caveats, thresholds, exclusions.
> 4. What comparable `<products/platforms>` actually do (observed patterns).
> 5. What *strengthens* `<enforceability / compliance / trust>` — the evidence/factors that matter.
>
> Flag where the law is state-dependent or ambiguous rather than asserting one rule. Do not fabricate citations.

This routinely surfaces scope you must *add* (e.g. a role you forgot, a mandatory party) and scope you can *cut* (things that are legally separate steps and out of scope).

---

## Phase 2 (greenfield variant) — landscape scan

> There's no codebase yet. Scan the landscape for `<domain/feature>`: comparable products and how they model `<the core concept>`, common architectures, the standard data model, the usual failure modes, and any dominant open-source building blocks. Prefer primary/official sources; cite them. Return a structured map I can design against — not a recommendation, a survey.
