# feature-design-pipeline

A reusable Claude Code **skill** that turns a vague, ambitious, or high-stakes feature ask into engineer-ready, codebase-grounded design docs — **before any code**. It's the workflow distilled from real end-to-end feature-scoping sessions (RentOk multi-party agreements): the sequence, the skills, the subagent patterns, and the rules that keep it honest.

> Design docs, not code. The pipeline ends by handing off to a plan (`planning-and-task-breakdown` / `writing-plans`), then build.

## What it does

Runs an ordered pipeline: **frame → ground the codebase → research → brief → design spec + flow diagrams → pre-mortem → lock decisions → adversarial doc review → persist → handoff.** It orchestrates other skills (it doesn't replace `brainstorming`, `brief`, `pre-mortem`) and adds the connective tissue they don't: parallel codebase maps, vendor/legal research, decision-locking, and persistence.

Two ideas do the heavy lifting:
1. **Ground before you design** — map the real code and verify the real vendor/legal facts first, in subagents, so raw bytes stay out of the window.
2. **Let reality reverse your locked decisions** — when a code map or doc contradicts a locked call, surface it; don't quietly design around a wrong call.

## Install

**As a personal skill** (per user):
```bash
cp -R feature-design-pipeline ~/.claude/skills/
```
Restart Claude Code (or start a new session). It appears in the skill list and auto-routes on asks like "design this feature", "scope this", "think through my ask and state it back", "write the brief/spec for X".

**As a project skill** (shared via a repo):
```bash
cp -R feature-design-pipeline <your-project>/.claude/skills/
```

**Invoke it** directly with the Skill tool / `/feature-design-pipeline`, or just start a feature the natural way and let the router pick it up.

## Files

| File | What |
|---|---|
| `feature-design-pipeline/SKILL.md` | The orchestrator — the phase pipeline, standing constraints, profiles, and the agent-skills backbone mapping. |
| `feature-design-pipeline/PLAYBOOK.md` | Human-readable playbook — the pipeline linearly with the *why* per phase. Read/share this. |
| `feature-design-pipeline/references/subagent-prompts.md` | Battle-tested subagent prompts (codebase map, vendor-API contract, legal/domain research, greenfield landscape). Adapt, don't reinvent. |
| `feature-design-pipeline/references/rentok-profile.md` | The RentOk overlay (conventions, paths, checkpoint recipe). **Swap this for your own team's profile** — see below. |

## Adapting it to your own system

The **generalized core** works for any project. The RentOk specifics live in `references/rentok-profile.md` (graphify-first, doc locations, `res.success`/paise conventions, team names, the memory/mem0/`gh` checkpoint). To use it in a different system, replace that file with your own profile:
- your codebase-map tool (graphify / `understand` / plain grep),
- your doc locations and API/DB conventions,
- your persistence recipe (where memory + ADRs live, which org repos go under),
- your mandatory chains.

The pipeline also branches **brownfield** (full code-map grounding) vs **greenfield** (landscape scan) automatically.

## Built on the agent-skills suite

Most of the pipeline's skills come from **[Addy Osmani's agent-skills](https://github.com/addyosmani/agent-skills)** (`brainstorming`, `ponytail`, `context-mode`, `understand`/`understand-domain`, `dispatching-parallel-agents`, `api-and-interface-design`, `documentation-and-adrs`, `planning-and-task-breakdown`, `source-driven-development`, `spec-driven-development`, `doubt-driven-development`, and more). Install that suite too for the full experience — the pipeline prefers a dedicated skill over hand-rolling a behavior. This skill is the *orchestration layer* on top.

## Provenance

The pipeline and its skill list were reconciled against three sources: the team's curated skill matrices, the raw session transcripts (what was *actually* invoked), and the installed agent-skills suite — trusting the transcripts where they disagreed. So it reflects what real sessions did, not an idealized list.
