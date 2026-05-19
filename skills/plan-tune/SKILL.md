---
name: plan-tune
description: Refine an existing /vcf-framework:plan output when it's close but not quite. Reads .vcf/<slug>/PLAN.md, asks targeted questions about weak spots, rewrites in place. Use when /vcf-framework:plan produced something 80% there, when Plan Mode output has obvious gaps, when the user says "tune the plan", "tighten the plan", "plan-tune", or runs /vcf-framework:plan-tune.
---

> Pattern adapted from `garrytan/gstack`'s `plan-tune` skill (refines arbitrary plans). Adapted for vcf-framework 2026-05-19 to operate on `.vcf/<slug>/PLAN.md`. Original concept: <https://github.com/garrytan/gstack>.

# /vcf-framework:plan-tune — Refine an existing PLAN.md

**Goal:** Tighten a plan that's almost right. Catches gaps without restarting Plan Mode from scratch.

## Argument

Takes one argument: `<feature-slug>`.

## Prereqs

- `<cwd>/.vcf/<feature-slug>/PLAN.md` must exist (output of `/vcf-framework:plan`).
- `<cwd>/.vcf/<feature-slug>/PRD.md` must exist.

If either missing, halt and route to `/vcf-framework:plan` or `/vcf-framework:prd`.

## Inputs to read

- `<cwd>/.vcf/<feature-slug>/PLAN.md` — the plan being tuned
- `<cwd>/.vcf/<feature-slug>/PRD.md` — acceptance criteria for goal-backward check
- `<cwd>/.vcf/<feature-slug>/design.md` — chosen shape + hard constraints

## Process

Walk through each section of PLAN.md, asking pointed questions about weak spots. Ask one at a time, wait for the user's answer, edit PLAN.md inline as decisions land.

### Weak-spot prompts

For each section of PLAN.md, ask:

**File tree changes**
- Are any of these files missing? (Cross-check against modules implied by PRD acceptance criteria — every criterion must trace to a file.)
- Are any of these files unnecessary?

**Module boundaries**
- Is every public interface explicitly typed?
- Are the consumers of each module enumerated?

**Data flow**
- For the non-trivial path, where is observability (logs / spans / metrics) wired?
- Where are the trust boundaries?

**Threat model**
- For each entry in PRD's scope (especially anything touching auth / multi-tenancy / payment / external input), is the mitigation named?
- For each input source, what's the validation layer?

**Phase breakdown (vertical slices)**
- Is each slice **end-to-end testable** (touches DB→DAL→API→UI→test)? If a slice is purely-DB or purely-UI, that's a horizontal layer — restructure.
- Are slice dependencies explicit?
- Can any slice ship in isolation as a meaningful increment?

**Verification strategy**
- For each slice, the test that proves it — does the test name exist as a placeholder in BUILD-NOTES.md, ready to be filled in `:build`?

## Output

Edit `<cwd>/.vcf/<feature-slug>/PLAN.md` in place. Add a `## Plan-tune log` section at the bottom:

```markdown
## Plan-tune log

Tuned by /vcf-framework:plan-tune at <ISO timestamp>.

### Changes
- <section> — <what was added / sharpened / removed> — <why>
- ...

### Risks flagged for /vcf-framework:build
- <risk> — <which slice it affects>
```

## After this step

1. Append to STATUS.md: `- plan-tune complete at <timestamp> → see PLAN.md updates`.
2. Tell the user verbatim: "Plan tuned for `<feature-slug>`. Run `/clear` then `/vcf-framework:build <feature-slug>` to start slice 1 (or whichever slice is next per `## Slice progress`)."

## Discipline

If plan-tune surfaces structural problems that need a re-plan (e.g., slices need to be re-cut entirely), don't try to fix that here. Halt and tell the user: "PLAN.md needs a re-plan, not a tune. Run `/clear` then `/vcf-framework:plan <feature-slug>` again."
