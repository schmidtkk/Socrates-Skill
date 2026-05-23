---
description: Teaching-mode synthesis pass over a prior Socratic session, with a clearly marked cold-draft fallback when no session exists. Re-derives the mechanism as a blog-style explanation for a smart reader new to this design, surfaces gaps interactively, and writes conclusion.html. Usage: /socratic:feynman [session-path]
argument-hint: [session-path]
---

Run the **socratic skill in FEYNMAN mode** (teaching-mode synthesis — see `skills/socratic/flows/feynman.md`).

Raw arguments: `$ARGUMENTS`

## What to do

1. Load the skill body at `skills/socratic/SKILL.md` and obey it.
2. Load `skills/socratic/flows/feynman.md` and every supporting file it explicitly references.
3. Parse `$ARGUMENTS`:
   - If a path is given → use that `.socrates/<ts>/` directory as the session.
   - If empty → detect from conversation context or latest `.socrates/<ts>/` on disk; if none exists, offer cold draft mode from user-provided context (see `skills/socratic/flows/feynman.md`).
4. Follow the FEYNMAN flow — including session detection, teaching-mode synthesis, interactive failure list, and draft review checkpoint.
5. Write `conclusion.html` to the resolved `.socrates/<ts>/` directory.

## What NOT to do

- Do not run a new Socratic question phase. This command is synthesis-only.
- Do not write outside `.socrates/<ts>/`.
- Do not stop the conversation mid-flow — use AskUserQuestion for all checkpoints and gap prompts.
