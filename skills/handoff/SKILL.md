---
name: handoff
description: Compact the current conversation into a handoff document for another agent to pick up. Use when the user runs /vcf-framework:handoff, says "hand off this session", "compact and write a pickup doc", or wants to save state between /clear boundaries during a long VCF loop.
argument-hint: "What will the next session be used for?"
---

> Originally from `mattpocock/skills` by Matt Pocock. Ported into vcf-framework 2026-05-19 under MIT. Original: <https://github.com/mattpocock/skills/tree/main/handoff>.

# Handoff

Write a handoff document summarising the current conversation so a fresh agent can continue the work. Save it to a path produced by `mktemp -t handoff-XXXXXX.md` (read the file before you write to it).

Suggest the skills to be used, if any, by the next session — especially the next vcf-framework gate skill (e.g., "Next session: `/clear` then `/vcf-framework:build my-feature`").

Do not duplicate content already captured in other artifacts (PRDs, plans, ADRs, issues, commits, diffs, `.vcf/<slug>/` state files). Reference them by path or URL instead.

If the user passed arguments, treat them as a description of what the next session will focus on and tailor the doc accordingly.

## VCF integration

When operating inside a vcf-framework loop, the handoff doc should include:

- Current feature slug (from `.vcf/<slug>/STATUS.md`)
- Current step in the loop and what's in-progress
- Pointer to the relevant `.vcf/<slug>/*.md` artifacts
- The next gate command the fresh agent should run after `/clear`
