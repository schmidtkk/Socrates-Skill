---
name: socratic
description: Socratic-Feynman method for mechanism exploration. Loads when the user invokes /socratic:ask, /socratic:think, or /socratic:feynman, or explicitly asks for "Socratic questioning", "Feynman synthesis", "mechanism grilling", "help me reveal the mechanism behind X", or "grill me on <topic> through mechanism questions". Do NOT auto-load for general Q&A, debugging, code review, or plan review.
---

# Socratic Skill

This skill is a two-phase discipline for understanding mechanisms:

1. **Socratic phase:** ask a few sharp, discriminative questions to expose the load-bearing gaps.
2. **Feynman phase:** re-articulate the mechanism in plain prose with visible warrants, evidence tags, and named gaps.

It is not a generic Q&A assistant, code reviewer, planner, debugger, or note-taking system.

## Command Dispatcher

| Command | Purpose | Required follow-up file |
|---|---|---|
| `/socratic:ask <concept|project> <topic>` | Grill the user one question at a time, then write `mechanism.md`. | `flows/ask.md` |
| `/socratic:think <concept|project> <topic>` | Self-grill, ask the user to review the question list once, run read-only probes, then write `mechanism.md`. | `flows/think.md` |
| `/socratic:feynman [session-path]` | Write a teaching-mode `conclusion.html` from a prior session, or a clearly marked cold draft if no session exists. | `flows/feynman.md` |

When a command runs, read the required flow file before acting. Load supporting files only when the flow references them.

## Core Invariants

These rules apply across all flows:

- **Topic modes:** ASK and THINK accept only `concept` and `project`. Do not invent `paper` or `task` modes.
- **Question budget:** target 3-5 grill questions; Q6-Q7 are allowed only for new high-value mechanism gaps; Q7 is the absolute cap.
- **Critic gate:** before asking or probing grill questions, generate at least 8 candidates and score them with `taste/critic-rubric.md`.
- **Taste:** keep only questions that are mechanism-revealing, uncertainty-reducing, discriminative, falsifiable, grounded, and actionable.
- **Interaction:** use AskUserQuestion for ritual questions, grill questions, review checkpoints, FEYNMAN failure prompts, and draft review. See `contracts/interaction.md`.
- **Probe safety:** auto-run only read-only probes. Side effects, network writes, code edits, and script execution become pending probes. See `contracts/artifacts.md`.
- **Artifacts:** write only under `.socrates/`. ASK and THINK write `mechanism.md`; FEYNMAN writes `conclusion.html`. See `contracts/artifacts.md`.
- **Verification:** every non-trivial synthesis claim carries a source tag: verified, user-asserted, unverified, or assumption. Narrative prose inherits `^v`, `^p`, `^u`, or `^a`.
- **Gap honesty:** never smooth over an unprobed section, missing artifact, contradiction, or unexplained term. Name it in the synthesis or failure list.

## Phase Boundary

Pre-synthesis is Socratic: ask, do not teach; keep alternatives alive; never mark candidate answers correct or recommended.

Post-grill synthesis is Feynman: teach plainly; thread the mechanism into a causal story; preserve verification tags, anti-alternatives, and ungrilled gaps.

## Supporting Files

| File | When to read |
|---|---|
| `flows/ask.md` | Running `/socratic:ask`. |
| `flows/think.md` | Running `/socratic:think`. |
| `flows/feynman.md` | Running `/socratic:feynman`. |
| `contracts/interaction.md` | Before any AskUserQuestion or checkpoint. |
| `contracts/artifacts.md` | Before writing artifacts, running probes, or tagging claims. |
| `synthesis/mechanism.md` | Before writing ASK/THINK `mechanism.md`. |
| `synthesis/conclusion-html.md` | Before writing FEYNMAN `conclusion.html`. |
| `taste/critic-rubric.md` | Before keeping, dropping, ordering, or reviewing candidate questions. |
| `taste/quality-axes.md` | When scoring candidate question quality. |
| `taste/paul-elder-types.md` | When assigning Socratic question type. |
| `taste/seeds/concept.md` | When the topic mode is `concept`. |
| `taste/seeds/project.md` | When the topic mode is `project`. |

## Out of Scope

- General Q&A or coding assistance.
- Plan/design review; use Plan mode or a dedicated grilling skill.
- Code review, refactoring, debugging, or task preflight.
- Persistent mechanism memory or question-pattern memory.
- Claude Code hooks.
- Writing outside `.socrates/`.
