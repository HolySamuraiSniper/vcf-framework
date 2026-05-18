---
name: overview
description: Loads the Vibe Coder Framework 10-step doctrine — tier system, anti-patterns, Joey playbook, and the difference between in-session mode and gate mode. Use when the user asks "what is VCF", "explain vibe coder framework", "vcf overview", "show me the tiers", "/vcf-framework:overview", or starts a new feature and needs to pick a tier. Loaded for reference, not workflow — does not write files or advance state.
---

# Vibe Coder Framework — Overview

A 10-step loop for moving from vague idea to shipped feature without the usual mid-flight chaos. The shape is **context-in → context-out**: every cycle starts by reading from memory + git history and ends by writing back what was learned, so the next cycle compounds. Two audit gates: **/vcf-framework:audit** (per-slice, during build) and **/vcf-framework:final-audit** (whole-feature, before closeout).

## Two execution modes

| Mode | When | How |
|---|---|---|
| **In-session** | Tier 1–2 momentum work, design is obvious, all in one chat | Read this overview, walk all 10 steps inline in the current conversation |
| **Gate mode** (default for Tier 3+) | Tier 3+ multi-session work | One slash command per gate step + `/clear` between every gate. Use `/vcf-framework:context`, `:brainstorm`, `:prd`, `:plan`, `:build`, `:audit`, `:final-audit`, `:closeout`. |

## Core philosophy

- **Ground first, code last.** Most "AI slop" comes from skipping context. The first 5 minutes spent reading prior decisions, recent commits, and surrounding code save hours of rework.
- **Tests before implementation, evals before "done".** Red/green proves the code runs. Evals prove it's actually correct on the dimensions that matter (semantics, perf, edge cases, regressions).
- **Independent reviewer, not the same context.** A second agent that hasn't seen your reasoning catches blind spots the original author can't.
- **100% complete, not MVP.** Ship a feature end-to-end — including docs, tests, telemetry, error states — rather than half-finished slices.
- **Named teammates over stateless subagents.** When work parallelizes, prefer `TeamCreate` with named teammates communicating via `SendMessage` over fire-and-forget subagent runs.

## When to enter the loop

Use this framework whenever the user signals non-trivial work:

- "Let's build / implement / add / refactor X"
- "How should I approach Y"
- "I need to ship Z by [date]"
- Any task spanning more than ~3 files or one obvious subsystem

For one-line bug fixes, typo edits, or pure questions, skip the loop and just do the work.

---

## The 10-step loop

### Step 01 — `/vcf-framework:context` — Ground the work

Search persistent memory for prior related decisions, gotchas, attempted approaches. Read `git log`, `git blame`, project `CLAUDE.md`/`AGENTS.md`. Write down three things that change your approach + any unverified assumptions. If you can't list three things, you didn't look hard enough.

### Step 02 — `/vcf-framework:brainstorm` — Vague idea → design doc

Use `superpowers:brainstorming` for the structured version. Capture: problem in the user's words, 2-3 alternative shapes, picked one + tradeoffs, hard constraints. A design doc you can't show another engineer is not a design doc.

### Step 03 — `/vcf-framework:prd` — Design doc → product requirements

Lock scope and acceptance criteria before architecture. Capture user stories, **concrete testable acceptance criteria** (these become red tests in build), explicit in/out scope (out > in), success metrics, non-goals.

### Step 04 — `/vcf-framework:plan` — PRD → technical architecture

**Use Plan Mode** (`EnterPlanMode` → `ExitPlanMode`) for any work that touches >3 files, crosses module boundaries, or matches the project's "Confirm approach before large changes" rule. Cover: file tree changes, module boundaries + interfaces, data flow, threat model paragraph, vertical-slice phase breakdown, per-slice verification strategy.

### Step 05 — `/vcf-framework:build` — Red → Green → Eval

One vertical slice per invocation. Use `superpowers:test-driven-development`. Write failing tests from acceptance criteria. Smallest change to green. Then evals: semantic correctness, perf vs PRD budget, edge cases, regression. Tests passing ≠ correct.

### Step 06 — `/vcf-framework:audit` — Independent reviewer (per build slice)

**Mandatory, not optional.** Pick invocation by tier:

| Tier | Invocation |
|---|---|
| 1–2 (tiny / small) | `Agent` tool with `subagent_type: "feature-dev:code-reviewer"` |
| 3 (medium, default) | `Skill` tool with `skill: "codex"` and `args: "review"` |
| 4 (grindy) | `Agent` tool with `subagent_type: "pr-review-toolkit:code-reviewer"` per chunk |
| 5 (security-critical) | `Skill` tool with `skill: "codex"` and `args: "challenge"` |

Give the reviewer **only the diff and the PRD**. Never paste your own reasoning.

### Step 07 — kaizen — Continuous refactoring (in-loop, not a gate)

After audit fixes are in, take 10–20 minutes for `v1.messy → v2.clean`. One commit per win — renames, extractions, dead code removal, comment cleanup, type tightening. If a commit takes >20 min, it's not kaizen — it's a refactor and belongs in its own loop.

### Step 08 — sprint — Parallelize within usage limits (in-loop, not a gate)

For solo work: chunks of 1–2 hours with verification at each boundary. For parallel work: `TeamCreate` with named teammates over stateless subagents. Phase progress visible: `phase 01 · auth ✓ | phase 02 · UI ▮▮▮▯ | phase 03 · data ▯▯▯▯`.

### Step 08.5 — `/vcf-framework:final-audit` — Comprehensive end-of-loop sweep

**Hard gate. Closeout never runs without it.** Per-slice audit catches local bugs; final audit catches integration drift no per-slice audit can see — partial migrations, half-removed scaffolding, dangling TODOs, telemetry that fires but is never read.

Run this chain in order — each step's output feeds the next:

1. **`/gsd-code-review`** — fresh-context reviewer over all files changed since context. Produces `REVIEW.md` with severity-tagged findings.
2. **`/codex review`** — cross-model independent check (OpenAI Codex). Tier 5 also: `/codex challenge`.
3. **`/gsd-verify-work`** — goal-backward verification against PRD acceptance criteria.
4. **`/gsd-code-review-fix`** — auto-applies fixes from REVIEW.md as atomic commits. Skip if zero P0/P1.

Tier 5 also: `Agent: pr-review-toolkit:silent-failure-hunter` + `Agent: gsd-security-auditor`. UI work also: `Skill: audit`.

### Step 09 — `/vcf-framework:closeout` — Docs, commit, ship, write back

Update README/docs/changelog. Final atomic commits with WHY (not WHAT). Ship via `superpowers:finishing-a-development-branch`, `/ship`, or `/land-and-deploy`. **Memory write-back** to claude-mem + Obsidian vault: surprises, decisions worth remembering, gotchas for next loop, new invariants. Without write-back, every loop starts from zero.

---

## When to skip steps

The loop is a default, not a law. Reasonable skips:

- **Trivial fix** (typo, one-line bug, version bump) — skip to build, then closeout
- **Pure exploration / spike** — context → brainstorm only, output is a doc, next loop makes it a PRD
- **Hot incident** — context → build (minimum fix) → audit → closeout (postmortem note)
- **Existing detailed spec from a stakeholder** — skip brainstorm and prd, go straight to plan

Steps that almost never skip:

- **context** — even one minute beats zero
- **audit** — independent reviewer catches what you can't on the slice
- **final-audit** — comprehensive sweep catches integration drift no per-slice audit can see
- **memory write-back at closeout** — compounding only happens if you actually feed it

## Anti-patterns

1. "I'll write the PRD after I have something working" → you'll write it to match the code, not the requirement
2. "I don't need to look at git log, I know this code" → recent commits often invalidate that knowledge
3. Briefing the reviewer with your reasoning → not independent, not useful
4. Bundling kaizen into the build commit → harder to review, harder to revert
5. Skipping memory write-back because the work is done → the next loop pays for it
6. Stateless `Agent` calls for multi-turn parallel work → use `TeamCreate` instead
7. Inline text + `AskUserQuestion` instead of `EnterPlanMode` → `ExitPlanMode` for the plan step → defeats the harness's review-before-build gate

---

## Joey playbook

When this framework runs on the Joey codebase (`/Users/tokiwilkinson/Projects/joey/`), use the substitutions and tier system below. The 10-step shape and discipline is unchanged — this just maps each step to Joey's specific tools.

### Tier system

| Tier | Trigger | Time | Process |
|---|---|---|---|
| **1 — Tiny** | Typo, copy tweak, 1-line bugfix, button color | 5–30 min | Skip framework, just code |
| **2 — Small** | 1 feature touching 1–3 files, design is obvious, <1 day | 1–3 hr | In-session: context (light) → build → closeout |
| **3 — Medium** ⭐ default | Multi-file feature, real design decisions, half-day to 1 day | 4–8 hr | Full gate mode, all 8 slash commands |
| **4 — Grindy** | 5+ similar items (per-provider onboarding, batch tests, translations) | day-shift design + overnight execution | Gate mode through `:plan`, then Ralph executes |
| **5 — Security-critical** | Touches auth, RLS, OAuth, payment, multi-tenancy | Tier 3 + ~30 min | Gate mode + `/codex challenge` at `:audit`, security extras at `:final-audit` |
| **6 — True milestone** | Multi-week, novel architecture (e.g., Phase 27 Episodic Memory) | Weeks | GSD outline + Tier 3 gate mode per vertical slice |

### Joey-specific skill substitutions

| VCF step | Joey substitution |
|---|---|
| **01 :context** | Read `CLAUDE.md` + `CONTEXT.md` + `docs/glossary.md`. `claude-mem:mem-search` for prior decisions. `git log -20` + `git blame` on files you'll touch. |
| **02 :brainstorm** | `superpowers:brainstorming` (unchanged) |
| **02.5 grill** (Joey addition, between brainstorm and prd) | `grill-with-docs` (or `grill-me-joey` for Joey-flavored tone). Converges after brainstorm, locks vocabulary into `CONTEXT.md` / glossary. |
| **03 :prd** | `to-prd` writes `.scratch/<feature>/PRD.md`, feeds `to-issues` downstream. |
| **04 :plan** | Plan Mode mandatory for multi-file. `superpowers:writing-plans` for smaller. |
| **05 :build** | `superpowers:test-driven-development` OR Joey's local `/tdd` skill. Pre-commit hook auto-validates. |
| **06 :audit** | Per tier table above. Security PR → `/codex challenge --background`. |
| **07 kaizen** | 10–20 min after audit fixes are in. One commit per win. |
| **08 sprint** | `TeamCreate` per CLAUDE.md mandate. `pnpm ralph` for grindy Tier-4 work. |
| **08.5 :final-audit** | Chain `/gsd-code-review` → `/codex review` → `/gsd-verify-work` → `/gsd-code-review-fix`. Tier 5 also: `pr-review-toolkit:silent-failure-hunter` + `gsd-security-auditor`. Post-auth UI also: `Skill: audit`. |
| **09 :closeout** | Atomic commits with WHY. Ship via `superpowers:finishing-a-development-branch` or `/ship`. **Memory write-back to claude-mem + vault wiki article via `obsidian-cli`.** |

### Joey gotchas to surface in every :context

- **FK guards** on cross-tenant inputs (test: `src/lib/ai/__tests__/fk-guard-import.test.ts`)
- **Schema budget** ceiling 4096 tokens (test: `src/lib/ai/__tests__/schema-budget.test.ts`)
- **Post-auth UI**: read `.stitch/DESIGN.md` + `.stitch/COMPONENTS.md` first
- **Multi-tenant**: ADR-0002 (RLS is the only barrier between tenants)
- **Glossary**: `docs/glossary.md` for canonical Danish↔English terms
- **Memory vocabulary** (v5.0, locked 2026-05-17 in CONTEXT.md): `Hukommelse` (capability), `Samtaleresumé` (Phase 27 row), `Hændelse` (Phase 28 row). Never say "a memory" in code.

### Tier 6 GSD hybrid (the milestone-only pattern)

For true milestones (Phase 27, future Phase 28, etc.), use GSD's lightweight outline + Tier 3 VCF per vertical slice. Skip GSD's heavy gate layer.

**Use from GSD:**

- `/gsd-new-milestone` — sets up the roadmap directory
- `/gsd-new-phase` — sets up the phase directory
- `/gsd-verify-work` — only at phase boundaries, not per slice
- `/gsd-complete-milestone` — at the end, archives the milestone

**Skip these GSD skills (replaced by lighter equivalents):**

- `/gsd-discuss-phase` → use `grill-with-docs` instead
- `/gsd-research-phase` → use direct Firecrawl / NotebookLM queries when actually needed
- `/gsd-plan-phase` → use Plan Mode + `to-prd` per vertical slice
- `/gsd-secure-phase` → use `/codex challenge` per security PR
- `/gsd-validate-phase` → pre-commit hook + tests handle this
- `/gsd-audit-fix`, `/gsd-audit-milestone` — overkill for solo work

Per vertical slice inside the phase, run Tier 3 gate mode. That's the day-to-day.

---

## The shape, one more time

Each loop is **context-in → context-out**. Step 01 reads from memory and history. Step 09 writes back to it. Skip Step 09's write-back and the loop is one-shot. Honor it and the system compounds.
