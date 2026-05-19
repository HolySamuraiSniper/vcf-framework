---
name: cleanup
description: Hygiene skill that archives or removes .vcf/<slug>/ state files after /vcf-framework:closeout ships. Keeps the .vcf/ directory tidy when running many loops. Use when the user runs /vcf-framework:cleanup, says "clean up this loop's state", "archive the .vcf entry", or wants to tidy up after a shipped feature.
---

> Pattern adapted from `gsd-build/get-shit-done`'s `gsd-cleanup` skill. Adapted for vcf-framework 2026-05-19 to archive `.vcf/<slug>/` state. Original concept: <https://github.com/gsd-build/get-shit-done>.

# /vcf-framework:cleanup — Archive a completed loop's state

**Goal:** Move a finished loop's `.vcf/<slug>/` directory to `.vcf/_archived/<slug>/` (or remove entirely) so the working `.vcf/` directory only contains active loops.

## Argument

Takes one argument: `<feature-slug>`.

## Prereqs

- `<cwd>/.vcf/<feature-slug>/STATUS.md` must show `**Current step:** COMPLETE` (output of `/vcf-framework:closeout`).
- `<cwd>/.vcf/<feature-slug>/CLOSEOUT.md` must exist.

If the loop isn't complete, halt and tell the user: "Cannot clean up `<feature-slug>` — STATUS.md shows it's not COMPLETE. Run `/clear` then `/vcf-framework:closeout <feature-slug>` first (or `/vcf-framework:forensics` if the loop failed)."

## Process

Ask the user the disposition for this loop's state files:

### Option A: Archive (default, recommended)

Move `.vcf/<slug>/` → `.vcf/_archived/<slug>/`. Preserves all state files in case future loops need to reference them, but gets them out of the active `.vcf/` directory.

```bash
mkdir -p .vcf/_archived
mv .vcf/<slug> .vcf/_archived/<slug>
```

### Option B: Compress

Tar + gzip into `.vcf/_archived/<slug>.tar.gz`, delete the directory. Saves disk space; harder to grep across.

```bash
mkdir -p .vcf/_archived
tar -czf .vcf/_archived/<slug>.tar.gz -C .vcf <slug>
rm -rf .vcf/<slug>
```

### Option C: Delete

Remove entirely. The loop's lessons are already in claude-mem + the Obsidian vault wiki (per `:closeout`); the local `.vcf/<slug>/` is redundant.

```bash
rm -rf .vcf/<slug>
```

Confirm with the user before doing Option C — it's not reversible.

## After this step

1. Verify the move/delete happened (`ls .vcf/_archived/` or `ls .vcf/` to confirm).
2. Tell the user: "Cleaned up `<feature-slug>`. Active loops remaining in `.vcf/`: [list]. Start your next loop with `/clear` then `/vcf-framework:context <new-feature-slug>`."

## Discipline

- **Never auto-cleanup.** Always wait for user confirmation, especially on Option C.
- **Never cleanup an in-progress loop.** The prereq check is load-bearing — if `:closeout` hasn't run, the state files are still active.
- **Don't touch `.vcf/_archived/`.** Once archived, files stay archived. If the user wants to retrieve a loop's history, they can `mv` back manually.

## When to skip

For Tier 1–2 momentum loops that run in one session, the user may prefer to leave `.vcf/<slug>/` in place as a memory aid rather than archiving. That's fine — cleanup is optional hygiene, not a gate.
