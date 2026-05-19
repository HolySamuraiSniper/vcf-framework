---
name: zoom-out
description: Zoom out and provide broader context or a higher-level perspective on a section of code. Use when the user is unfamiliar with a section of code, says "zoom out", "give me a map", "what calls this", "show me the architecture here", or runs /vcf-framework:zoom-out. Strengthens /vcf-framework:context when starting work in an unfamiliar module.
---

> Originally from `mattpocock/skills` by Matt Pocock. Ported into vcf-framework 2026-05-19 under MIT. Original: <https://github.com/mattpocock/skills/tree/main/zoom-out>.

# Zoom Out

I don't know this area of code well. Go up a layer of abstraction. Give me a map of all the relevant modules and callers, using the project's domain glossary vocabulary.

## VCF integration

When invoked inside a vcf-framework loop:

1. Read the project's `CONTEXT.md` + `docs/glossary.md` (or equivalent) first — use the domain vocabulary the project already defines.
2. Produce the map at a level of abstraction the caller actually needs (a single module's callers, a subsystem's modules, or the whole feature area).
3. If invoked from `/vcf-framework:context` for a specific module, append your output to `<cwd>/.vcf/<feature-slug>/CODEBASE-MAP.md` under a `## Zoom-out (<module path>)` section. Never overwrite — `map-codebase` writes subsystem-level overviews into the same file and your section coexists with theirs.
