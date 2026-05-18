---
name: final-audit
description: VCF gate-mode Step 08.5 — comprehensive end-of-loop sweep over the whole feature, not just the last slice. Chains /gsd-code-review → /codex review → /gsd-verify-work → /gsd-code-review-fix. Tier 5 adds pr-review-toolkit:silent-failure-hunter + gsd-security-auditor. UI work adds /audit. Reads all .vcf/<feature-slug>/ artifacts. Writes FINAL-REVIEW.md and VERIFY.md. Use when the user runs /vcf-framework:final-audit, says "vcf final audit X", "comprehensive review of X", or has all slices built and per-slice audited and is ready for the gate before /vcf-framework:closeout. This gate is mandatory before closeout.
---

# /vcf-framework:final-audit — Step 08.5: Comprehensive end-of-loop sweep

**Goal:** Before closeout, audit the **entire feature as one shipped thing** against the PRD — not just the last slice.

**This is a hard gate. `/vcf-framework:closeout` never runs without it.**

Per-slice audit catches local bugs; final audit catches integration drift no per-slice audit can see — partial migrations, half-removed scaffolding, dangling TODOs, telemetry that fires but is never read, slice 1 → slice 3 interactions nobody tested.

This is Step 08.5 of 10 in VCF gate mode. For full doctrine, see `vcf-framework:overview`.

## Argument

Takes one argument: `<feature-slug>`.

## Prereqs

- Every slice in `PLAN.md` must be marked `audited` in `STATUS.md`'s `## Slice progress` section.
- All `REVIEW-slice-*.md` files must show zero open P0 findings.

If any slice is `pending`, `built — awaiting audit`, or `audit p0 fixing`, **halt** and route the user back to `/vcf-framework:build` or `/vcf-framework:audit`.

## Inputs to read

- All `<cwd>/.vcf/<feature-slug>/*.md` artifacts — full loop context
- `<cwd>/.vcf/<feature-slug>/PRD.md` — acceptance criteria for goal-backward verification
- `<cwd>/.vcf/<feature-slug>/PLAN.md` — threat model section for security check
- `git diff` from the start of the loop to HEAD — the full body of work
- `<cwd>/.vcf/<feature-slug>/STATUS.md` — to confirm tier (for Tier 5 extras)

## Process

Run this chain in order. Each step's output feeds the next.

### 1. `/gsd-code-review`

Invoke via `Skill` tool with `skill: "gsd-code-review"`. Spawns a fresh-context reviewer over all files changed since the loop started. Produces `REVIEW.md` with severity-tagged findings.

Move or copy `REVIEW.md` to `<cwd>/.vcf/<feature-slug>/FINAL-REVIEW.md`.

### 2. `/codex review`

Invoke via `Skill` tool with `skill: "codex"` and `args: "review"`. Cross-model independent check from OpenAI Codex CLI. Append its findings to `FINAL-REVIEW.md` under a `## Codex review` section.

For Tier 5 (security-critical), **also** run `Skill: codex args: challenge` for adversarial pressure. Append output as `## Codex challenge` section.

### 3. `/gsd-verify-work`

Invoke via `Skill` tool with `skill: "gsd-verify-work"`. Goal-backward verification — does the shipped diff actually deliver every acceptance criterion from `PRD.md`? Not "did tests pass" — "is the feature *real*".

Write its output to `<cwd>/.vcf/<feature-slug>/VERIFY.md`.

### 4. `/gsd-code-review-fix`

Invoke via `Skill` tool with `skill: "gsd-code-review-fix"`. Auto-applies fixes from `FINAL-REVIEW.md` as atomic commits, one per fix. Produces `REVIEW-FIX.md`.

**Skip if `FINAL-REVIEW.md` has zero P0 / zero P1 findings.**

Copy resulting `REVIEW-FIX.md` to `<cwd>/.vcf/<feature-slug>/FINAL-REVIEW-FIX.md`.

### 5. (Tier 5 only) Security extras

- `Agent` tool with `subagent_type: "pr-review-toolkit:silent-failure-hunter"` — hunts swallowed errors, inadequate fallbacks, catch blocks that hide bugs. Append findings to `FINAL-REVIEW.md` under `## Silent failure hunt`.
- `Agent` tool with `subagent_type: "gsd-security-auditor"` — verifies threat-model mitigations from `PLAN.md`'s threat-model section actually exist in shipped code. Write its output to `<cwd>/.vcf/<feature-slug>/SECURITY.md`.

### 6. (Frontend / UI work only) Quality sweep

`Skill` tool with `skill: "audit"` — technical quality sweep across a11y, performance, theming, responsive, anti-patterns with P0–P3 scoring. Append findings to `FINAL-REVIEW.md` under `## UI quality audit`.

## Discipline

If Step 06 (per-slice audits) caught severe issues but Step 08.5 (final) doesn't, your reviewer was briefed too narrowly — re-run with broader scope.

If Step 08.5 finds issues every Step 06 missed, that's signal. Log them in `/vcf-framework:closeout` memory write-back as a slice-audit calibration note — pattern, what slice-audit missed, what would have caught it. The framework improves over time only if you feed back what was missed.

**Per-slice audit catches local bugs; final audit catches the integration drift that no per-slice audit can see.**

## After this step

If `FINAL-REVIEW.md` or `VERIFY.md` surface unresolved P0/P1: fix them. Likely a new `/vcf-framework:build` + `/vcf-framework:audit` cycle on a targeted slice. Do not proceed to closeout with open P0/P1.

Otherwise:

1. Update `STATUS.md`:
   - Append `- 08.5 complete at <timestamp> → FINAL-REVIEW.md, VERIFY.md` + (FINAL-REVIEW-FIX.md if step 4 ran) + (SECURITY.md if step 5 ran)
   - Update `**Current step:**` to `09 — /vcf-framework:closeout`
   - Update `**Last completed:**` to `08.5`
2. Tell the user verbatim: "Final audit passed for `<feature-slug>`. Run `/clear` then `/vcf-framework:closeout <feature-slug>` to ship and write back."
