---
description: Self-grill with one user review checkpoint after question selection; produce a verifiable mechanism model at .socrates/<ts>/mechanism.md. Usage: /socratic:think <concept|project> <topic>
argument-hint: <concept|project> <topic>
---

Run the **socratic skill in THINK mode** (self-grill with one MANDATORY user review checkpoint after the question list is drafted — see `skills/socratic/flows/think.md`).

Raw arguments: `$ARGUMENTS`

## What to do

1. Load the skill body at `skills/socratic/SKILL.md` and obey it.
2. Load `skills/socratic/flows/think.md` and every supporting file it explicitly references.
3. Parse `$ARGUMENTS` as: `<topic-mode> <topic>`
   - `topic-mode` MUST be one of: `concept` | `project`
4. Follow the THINK flow — including the **mode-split startup**:
   - `concept` → think-then-fetch (generate candidates from training data, fetch authoritative sources during probe phase)
   - `project` → fetch-then-think (read README/CLAUDE.md, glob top-level, grep key symbols *first*, then generate candidates)

## If arguments invalid

If `$ARGUMENTS` is empty, or topic-mode is not `concept`/`project`, respond with exactly:

```
Usage: /socratic:think <concept|project> <topic>

Examples:
  /socratic:think concept "diffusion guidance"
  /socratic:think project src/auth
  /socratic:think project .
```

…and stop. Do not start the self-grill.
