# vcf-framework

A Claude Code plugin that wraps the **Vibe Coder Framework** — a 10-step doctrine with 8 gate commands — for shipping production-quality features end-to-end. Each gate is its own slash command, reads its upstream artifact, writes its own output, and tells you to `/clear` before the next gate. Two in-loop sub-steps (kaizen, sprint) live inside `:build` and aren't gate commands.

This is what GSD looks like for solo / small-team development without the milestone-level overhead.

## Two execution modes

Pick the mode by tier (see `/vcf-framework:overview` for tier definitions):

| Mode | When | How |
|---|---|---|
| **In-session** (`/vcf-framework:overview` reads the doctrine, then you stay in one chat) | Tier 1–2 momentum work | One conversation, all 10 steps inline |
| **Gate mode** (this plugin's primary use) | Tier 3+ work | One slash command per step + `/clear` between every gate |

## The 8 gate commands

In order. Each refuses to run if its upstream prereq file is missing in `.vcf/<feature-slug>/`.

| Command | Step | Reads | Writes |
|---|---|---|---|
| `/vcf-framework:context <slug>` | 01 | memory, git history, CLAUDE.md, AGENTS.md | `CONTEXT.md`, bootstraps `STATUS.md` |
| `/vcf-framework:brainstorm <slug>` | 02 | `CONTEXT.md` | `design.md` |
| `/vcf-framework:prd <slug>` | 03 | `design.md` | `PRD.md` |
| `/vcf-framework:plan <slug>` | 04 | `PRD.md` | `PLAN.md` (via Plan Mode for multi-file work) |
| `/vcf-framework:build <slug>` | 05 | `PLAN.md`, current slice in `STATUS.md` | appends to `BUILD-NOTES.md` |
| `/vcf-framework:slice-audit <slug>` | 06 | `BUILD-NOTES.md`, `PRD.md` | `REVIEW-slice-N.md` |
| `/vcf-framework:final-audit <slug>` | 08.5 | everything | `FINAL-REVIEW.md`, `VERIFY.md` |
| `/vcf-framework:closeout <slug>` | 09 | `FINAL-REVIEW.md`, `VERIFY.md` | `CLOSEOUT.md`, ships, writes back to memory |

Plus one reference skill: `/vcf-framework:overview` — the 10-step doctrine, tier system, and Joey playbook. Read this once when first using the plugin.

(Steps 07 `kaizen` and 08 `sprint` are intentionally not gate commands — they're tactical sub-loops inside `/vcf-framework:build`. See the overview for guidance.)

## v0.2 — Cherry-picked utilities (15 skills)

In addition to the 8 gate commands + overview, v0.2 ships 15 cherry-picked skills from gstack, GSD, Pocock, and Karpathy. Each integrates with the gate flow at specific moments.

### Ported skills (verbatim with attribution)

| Skill | Source | When to use |
|---|---|---|
| `/vcf-framework:handoff` | Pocock | Compact a session into a fresh-agent pickup doc. Pairs with `/clear`. |
| `/vcf-framework:zoom-out` | Pocock | Module-map an unfamiliar area. Strengthens `:context`. |
| `/vcf-framework:prototype` | Pocock | Throwaway prototype between `:brainstorm` and `:prd`. |
| `/vcf-framework:diagnose` | Pocock | Disciplined hard-bug loop during `:build`. |
| `/vcf-framework:improve-codebase-architecture` | Pocock | Refactor mode between loops. |
| `/vcf-framework:write-a-skill` | Pocock | Add a new skill to vcf-framework itself. |
| `/vcf-framework:setup-precommit` | Pocock | One-shot bootstrap for fresh repos (Husky + lint-staged + typecheck + test). |
| `/vcf-framework:grill-with-docs` | Pocock | Rigorous design interview — alternative to `:brainstorm` for novel features. |
| `/vcf-framework:karpathy-guidelines` | Karpathy / multica-ai | 4 principles discipline overlay during `:build` or `:slice-audit`. |
| `/vcf-framework:forensics` | GSD (adapted) | Post-mortem for FAILED or ABANDONED loops. |

### Adapted skills (vcf-flavored versions of external patterns)

| Skill | Inspired by | When to use |
|---|---|---|
| `/vcf-framework:retro` | gstack | Structured retrospective ritual before `:closeout`. |
| `/vcf-framework:plan-tune` | gstack | Refine an existing `PLAN.md` that's close but not quite. |
| `/vcf-framework:office-hours` | gstack | Pre-`:brainstorm` ideation conversation. |
| `/vcf-framework:map-codebase` | GSD | Architectural overview of a large unfamiliar codebase. |
| `/vcf-framework:cleanup` | GSD | Archive `.vcf/<slug>/` state after `:closeout` ships. |

### Patterns baked into gate skills (no new files)

- `:overview` — Tier-6 milestone hybrid (GSD + VCF combo for true milestones)
- `:plan` — Goal-backward plan-check before exiting Plan Mode
- `:build` — Karpathy discipline check + wave-based parallel slices for Tier-4
- `:slice-audit` — Karpathy 4-principle check during review
- `:final-audit` — Self-eval Step 0 pre-check (alirezarezvani-engineering pattern)
- `:closeout` — Dangling-threads section + tech-debt-tracker reference

### References (external skills the framework points at)

- Pocock `to-prd` from `:prd` (alternative entry path)
- Pocock `to-issues` from `:plan` (alternative slice breakdown)
- gstack `design-shotgun` from `:plan` and `:build` (UI variant generation)
- alirezarezvani `focused-fix` from `:build` (bug-fix scope discipline)
- alirezarezvani `autoresearch-agent` from `:closeout` (Karpathy autoresearch loops)
- alirezarezvani `tech-debt-tracker` from `:closeout` dangling-threads section
- alirezarezvani `karpathy-coder` from `:karpathy-guidelines` (heavyweight enforcement)

## State convention

Each invocation reads/writes files in `<cwd>/.vcf/<feature-slug>/`:

```
<cwd>/.vcf/<feature-slug>/
├── STATUS.md          # Loop tracker — single source of truth for "where am I"
├── CONTEXT.md         # output of /vcf-framework:context
├── design.md          # output of /vcf-framework:brainstorm
├── PRD.md             # output of /vcf-framework:prd
├── PLAN.md            # output of /vcf-framework:plan
├── BUILD-NOTES.md     # appended by /vcf-framework:build per slice
├── REVIEW-slice-N.md  # output of /vcf-framework:slice-audit per slice
├── FINAL-REVIEW.md    # output of /vcf-framework:final-audit
├── VERIFY.md          # goal-backward check from /vcf-framework:final-audit
└── CLOSEOUT.md        # output of /vcf-framework:closeout
```

Add `.vcf/` to your repo's `.gitignore` unless you want to commit framework state alongside code.

## Typical flow

```
/vcf-framework:context my-feature
/clear
/vcf-framework:brainstorm my-feature
/clear
/vcf-framework:prd my-feature
/clear
/vcf-framework:plan my-feature
/clear
/vcf-framework:build my-feature        # slice 1
/clear
/vcf-framework:slice-audit my-feature        # review slice 1
/clear
/vcf-framework:build my-feature        # slice 2
/clear
/vcf-framework:slice-audit my-feature        # review slice 2
... (repeat per slice in PLAN.md)
/vcf-framework:final-audit my-feature  # whole-feature sweep
/clear
/vcf-framework:closeout my-feature     # ship + memory write-back
```

Each `/clear` resets context. Each gate skill reads only the upstream artifact, not your chat history. Total context budget per gate stays bounded — the secret to long-running features without conversation cruft.

## Installation

```bash
claude plugin marketplace add HolySamuraiSniper/vcf-framework
claude plugin install vcf-framework@vcf-framework
```

Verify the install:

```bash
claude plugin details vcf-framework@vcf-framework
```

You should see 9 skills (v0.1) or 24 skills (v0.2+) in the component inventory. In a fresh Claude Code session, type `/vcf-framework:overview` to load the doctrine reference.

### Local development install

If you're hacking on the plugin itself:

```bash
git clone https://github.com/HolySamuraiSniper/vcf-framework.git
cd vcf-framework
claude plugin marketplace add .
claude plugin install vcf-framework@vcf-framework
```

## Tier guide (from /vcf-framework:overview)

| Tier | Trigger | Mode |
|---|---|---|
| 1 — Tiny | Typo, 1-line bugfix | Skip framework, just code |
| 2 — Small | 1 feature, 1–3 files, design obvious | In-session: `/vcf-framework:context` → `/vcf-framework:build` → `/vcf-framework:closeout` |
| **3 — Medium** ⭐ | Multi-file, real decisions, half-day to 1 day | Gate mode, full 8-command sequence |
| 4 — Grindy | 5+ similar items (batch work) | Design once (gate mode through `:plan`), then Ralph executes |
| 5 — Security-critical | Auth, RLS, OAuth, payment, multi-tenancy | Gate mode + `/codex challenge` at `:slice-audit`, security extras at `:final-audit` |
| 6 — True milestone | Multi-week, novel architecture | GSD outline + Tier 3 VCF per vertical slice |

## What the framework guarantees

- **No PRD drift**: scope is locked at `:prd`, audited against at `:final-audit`
- **No reviewer brief contamination**: every audit invocation runs in fresh context (Agent tool) or sub-agent (Skill tool wrapping `/codex`)
- **Two audit gates**: per-slice (`:slice-audit`) catches local bugs; whole-feature (`:final-audit`) catches integration drift
- **Memory write-back**: `:closeout` always feeds claude-mem + Obsidian vault. The loop compounds.

## What it doesn't do

- Replace GSD for true Tier-6 milestones. Use GSD's milestone scaffolding (`/gsd-new-milestone`, `/gsd-new-phase`) plus this plugin per vertical slice.
- Auto-detect tier. You pick. (Add tier explicitly to `STATUS.md` after `:context`.)
- Pick model defaults. Provider config lives in your project's own config.

## Author

Toki Wilkinson · [@HolySamuraiSniper](https://github.com/HolySamuraiSniper)

Built with Claude Code · MIT licensed
