---
name: slice-audit
description: VCF gate-mode Step 06 — independent per-slice reviewer that runs in fresh context (not the build agent's context). Picks invocation by tier — feature-dev:code-reviewer for T1-2, /codex review for T3, pr-review-toolkit:code-reviewer for T4, /codex challenge for T5. Reads BUILD-NOTES.md and PRD.md. Writes .vcf/<feature-slug>/REVIEW-slice-N.md. Use when the user runs /vcf-framework:slice-audit, says "vcf slice-audit X", "vcf audit slice N", "review slice N", or has completed a /vcf-framework:build slice and needs per-slice review before the next slice or before /vcf-framework:final-audit.
---

# /vcf-framework:slice-audit — Step 06: Independent reviewer (per build slice)

**Goal:** A second pair of eyes that hasn't seen your reasoning, looking specifically for what the original author can't see *on the slice just built*.

This is Step 06 of 10 in VCF gate mode. **Mandatory, not optional** — every built slice gets an audit before the next slice starts or before `/vcf-framework:final-audit`.

## Argument

Takes one argument: `<feature-slug>`.

## Prereqs

- `<cwd>/.vcf/<feature-slug>/BUILD-NOTES.md` must show at least one slice with status `built — awaiting audit` in STATUS.md.
- `<cwd>/.vcf/<feature-slug>/PRD.md` must exist.

If no slice is awaiting audit, **halt** and tell the user: "No slice awaiting audit for `<feature-slug>`. Either run `/vcf-framework:build <feature-slug>` first, or if all slices are audited, run `/vcf-framework:final-audit <feature-slug>`."

## Inputs to read

- `<cwd>/.vcf/<feature-slug>/BUILD-NOTES.md` — most recent slice section
- `<cwd>/.vcf/<feature-slug>/PRD.md` — acceptance criteria the slice claims to satisfy
- `<cwd>/.vcf/<feature-slug>/STATUS.md` — identify the slice number being audited + the tier

## Process

### 1. Identify the slice and tier

Read STATUS.md. Extract:

- The slice currently marked `built — awaiting audit` in `## Slice progress` (this is the slice you'll audit).
- The `**Tier:**` value from the header (1, 2, 3, 4, or 5).

**If `**Tier:**` is `UNSET` or missing:** halt without invoking any reviewer. Tell the user verbatim: "Cannot pick reviewer for `<feature-slug>` — Tier is UNSET in STATUS.md. Set `**Tier:**` to 1, 2, 3, 4, or 5 per `/vcf-framework:overview` tier table, then re-run `/vcf-framework:slice-audit <feature-slug>`."

### 2. Pick the reviewer invocation by tier

| Tier | Invocation | Why this one |
|---|---|---|
| 1–2 (tiny / small) | `Agent` tool with `subagent_type: "feature-dev:code-reviewer"` | Confidence-filtered — only surfaces high-priority findings, no nits |
| 3 (medium, default) | `Skill` tool with `skill: "codex"` and `args: "review"` | Independent cross-model review via OpenAI Codex CLI |
| 4 (grindy) | `Agent` tool with `subagent_type: "pr-review-toolkit:code-reviewer"` per chunk | Style / convention check on each chunk |
| 5 (security-critical) | `Skill` tool with `skill: "codex"` and `args: "challenge"` | Adversarial mode — Codex actively tries to break the code |

**Whichever invocation you use:** give the reviewer **only the diff and the PRD acceptance criteria**. Never paste your own reasoning. Never paste BUILD-NOTES.md (that's your own thinking — defeats the purpose).

`Skill`-invoked reviewers (`/codex`) run in your context but spawn their own sub-agent with fresh context internally. `Agent`-invoked reviewers get fresh context for free.

What the reviewer looks for:

- **Logic bugs** that look right but aren't
- **Security issues** — input validation, secrets in logs, auth boundaries, tenant isolation
- **Bad patterns** that work but will rot — god functions, leaky abstractions, missing error paths
- **Mismatch between diff and PRD** — does the code actually do what the acceptance criteria say?

## Output

Write `<cwd>/.vcf/<feature-slug>/REVIEW-slice-<N>.md`:

```markdown
# Audit: Slice <N> — <ISO timestamp>

Reviewer: <tier-appropriate tool used>
Slice name: <from PLAN.md>

## P0 — must fix before next slice

- ...

## P1 — should fix before /vcf-framework:final-audit

- ...

## P2 — nice to have

- ...

## P3 — defer / noted

- ...

## Mismatch with PRD?

- <yes/no + details. If yes, list which acceptance criteria are not actually satisfied by the diff.>

## Notes for /vcf-framework:final-audit

- <anything the per-slice reviewer flagged that the whole-feature audit should re-check>
```

## Discipline

The point of the audit is the **independence**. If you brief the reviewer with your reasoning, you've defeated the purpose. Save your reasoning for /vcf-framework:closeout's memory write-back.

**Do not skip on "small" slices.** Small slices accumulate, and the integration audit (`/vcf-framework:final-audit`) can't catch slice-local logic errors — it audits the whole picture, not the individual brushstroke.

## After this step

If P0 findings: fix them in a new `/vcf-framework:build` cycle on the same slice. Don't proceed to the next slice with P0 open. Update STATUS.md slice status to `built — audit p0 fixing`.

If only P1+P2+P3 findings (no P0):

1. Update `STATUS.md`:
   - Change current slice in `## Slice progress` from `built — awaiting audit` to `audited (P0: 0, P1: N, P2: N, P3: N)`
   - Append `- 06 slice <N> audited at <timestamp> → see REVIEW-slice-<N>.md` to step log
2. If more slices remain in PLAN.md: update `**Current step:**` to `05 — /vcf-framework:build (slice <N+1>)` and tell the user: "Slice <N> audited. Run `/clear` then `/vcf-framework:build <feature-slug>` for slice <N+1>."
3. If all slices are audited: update `**Current step:**` to `08.5 — /vcf-framework:final-audit` and tell the user: "All slices built and audited. Run `/clear` then `/vcf-framework:final-audit <feature-slug>` for the comprehensive sweep."
