---
name: build
description: VCF gate-mode Step 05 — executes one vertical slice from PLAN.md with Red → Green → Eval discipline. Reads PLAN.md and STATUS.md to find current slice. Appends to .vcf/<feature-slug>/BUILD-NOTES.md. Use when the user runs /vcf-framework:build, says "vcf build X", "implement slice N of X", or has completed :plan and is starting implementation. Runs once per vertical slice; expect to be invoked multiple times with /clear and /vcf-framework:slice-audit between each.
---

# /vcf-framework:build — Step 05: Red → Green → Eval

**Goal:** Write tests that fail. Make them pass. Then prove the result is actually correct, not just non-crashing.

This is Step 05 of 10 in VCF gate mode. **One vertical slice per invocation.** For full doctrine, see `vcf-framework:overview`.

## Argument

Takes one argument: `<feature-slug>`.

## Prereqs

- `<cwd>/.vcf/<feature-slug>/PLAN.md` must exist (output of `/vcf-framework:plan`).
- `<cwd>/.vcf/<feature-slug>/STATUS.md` must have a `## Slice progress` section.

If `PLAN.md` is missing, **halt** and route the user back to `/vcf-framework:plan`.

## Inputs to read

- `<cwd>/.vcf/<feature-slug>/PLAN.md` — phase breakdown, file tree, verification strategy
- `<cwd>/.vcf/<feature-slug>/PRD.md` — acceptance criteria (these become red tests)
- `<cwd>/.vcf/<feature-slug>/STATUS.md` — identify current slice from `## Slice progress`
- `<cwd>/.vcf/<feature-slug>/BUILD-NOTES.md` if it exists — see what prior slices did

## Process

### 1. Identify the current slice

From STATUS.md `## Slice progress`, find the first slice marked `pending`. If all slices are complete, halt and tell the user to run `/vcf-framework:final-audit` instead.

### 2. Red

For each acceptance criterion in `PRD.md` that this slice covers, write a failing test. Use `superpowers:test-driven-development` for the red/green discipline.

The test failing **for the right reason** is the proof you understood the requirement. If the test fails for the wrong reason (e.g., import error instead of assertion error), fix the test first.

### 3. Green

Smallest change that makes the test pass. Resist the urge to "fix the surrounding code while I'm in here" — that's kaizen (Step 07), a separate commit *after* `/vcf-framework:slice-audit`.

Commit as soon as green. One commit per slice (or a small chain if the slice has internal phases). Commit messages explain WHY, not WHAT.

**For narrow bug-fix slices**, also invoke `focused-fix` (alirezarezvani-engineering bundle) discipline — fix only what was asked, don't expand scope into related-but-different issues. The skill enforces the "every changed line traces to the reported bug" rule.

**For UI / frontend slices**, also consider invoking `/design-shotgun` (gstack) before the Green step — generates multiple visual variants so you commit the right shape, not the first shape.

### Karpathy discipline check (before committing)

Before committing the slice, run the Karpathy 4-principle check from `/vcf-framework:karpathy-guidelines`. Especially:

- **Surgical Changes**: does every changed line trace to a PRD acceptance criterion? If not, that line is scope creep — remove it.
- **Simplicity First**: if you wrote 200 lines and 50 would do, rewrite.
- **Think Before Coding**: did you state assumptions explicitly anywhere in the slice? Or did you silently pick interpretations?

The check is cheap. The 4 principles are at `skills/karpathy-guidelines/SKILL.md`.

### Wave-based parallel slices (Tier-4 only)

For Tier-4 grindy work where 2+ slices in PLAN.md are **mutually independent** (no shared files, no FK dependencies, no read-after-write between them), build them in parallel:

1. Use `TeamCreate` to spin up named teammates: `slice-1-builder`, `slice-2-builder`, etc.
2. Each teammate runs Red → Green → Eval on its own slice in isolation.
3. Teammates communicate progress via `SendMessage` if they hit cross-slice surprises.
4. After all parallel slices commit, audit each one separately via `/vcf-framework:slice-audit` — never batch audits, the per-slice independence is the point.

If slices share state or have ordering constraints, **do not parallelize** — sequential is correct.

### 4. Eval

Tests passing ≠ correct. After green, run evals on the dimensions that actually matter:

- **Semantic correctness** — does the output mean what it should? (For LLM features: does the response actually answer the question? For data pipelines: does the row count and shape match expectations?)
- **Performance** — does it fit the budget from PRD?
- **Edge cases** — empty inputs, concurrent calls, malformed data, timezones, off-by-one boundaries, the things from the PRD's edge-case bullets
- **Regression** — did anything else break? Run the broader test suite.

If evals fail, that's a red and you go back to green. The eval layer is what separates "the tests pass" from "the feature works".

## Output

**Append** to `<cwd>/.vcf/<feature-slug>/BUILD-NOTES.md` (create if it doesn't exist yet):

```markdown
## Slice <N>: <name> — <ISO timestamp>

### Red
- Tests written: <test names + paths>
- Failing for the right reason: <yes / no + diagnosis if no>

### Green
- Files changed: <list>
- Commit(s): <sha + one-line message>

### Eval
- **Semantic:** <pass / fail + notes>
- **Performance:** <measurement vs PRD budget>
- **Edge cases:** <which checked + results>
- **Regression:** <test suite result — e.g. 854/854 green>

### Open issues to surface in /vcf-framework:slice-audit
- ...
```

## Discipline

The **eval layer** is what separates "tests pass" from "feature works". Skip eval and you've shipped slop with a green CI badge.

**One slice per invocation.** Don't try to build slice 2 inside the same `/vcf-framework:build` call. Each slice gets `/clear` and `/vcf-framework:slice-audit` between, because slice-level audit is what catches local bugs before they compound.

## After this step

1. Update `STATUS.md`:
   - Change current slice in `## Slice progress` from `pending` to `built — awaiting audit`
   - Append `- 05 slice <N> complete at <timestamp> → see BUILD-NOTES.md` to step log
   - Update `**Current step:**` to `06 — /vcf-framework:slice-audit (slice <N>)`
2. Tell the user verbatim: "Slice <N> built. Run `/clear` then `/vcf-framework:slice-audit <feature-slug>` to review this slice before starting the next."
