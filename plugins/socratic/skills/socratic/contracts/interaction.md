# Interaction Contract

Use AskUserQuestion for every structured user touchpoint. The only open-text moments are the mandatory ASK-mode "why" follow-up and parameter follow-ups after a structured checkpoint choice.

## ASK Grill Questions

Each grill question must use AskUserQuestion with:

- One question sentence.
- 3-4 selectable candidate answers.
- Auto-added Other.
- This tail line:

```text
You can also pick **Other** and type `skip` (skip this question), `enough` (synthesize now), or `pivot` (regenerate questions from new angle).
```

### Candidate Answer Rules

MUST:
- Each option is a different mechanism hypothesis.
- Each option is plausible by some reasonable interpretation.
- Options cover the realistic answer space.
- Options are roughly equally specific.

MUST NOT:
- Provide one obviously correct option.
- Mark an option correct or recommended.
- Provide fewer than 3 or more than 4 options, excluding Other.
- Teach through the options. Teaching starts only in synthesis.

### Control Tokens

If Other contains a control token, handle it before asking "why":

| Token | Action |
|---|---|
| `skip` | Record a control event and move to the next kept candidate. |
| `enough` | Discard remaining candidates and synthesize. |
| `pivot` | Record a control event, regenerate candidates from the new angle, and continue. Pivot consumes 1 question budget. |

For control-token responses, do not ask the mandatory follow-up and do not append a normal `{question, answer, why}` transcript item.

For normal answers, always ask the user why they picked that option, then append `{question, answer, why, other_used}` to `transcript.md`.

## THINK Review Checkpoint

THINK mode has one mandatory review checkpoint after candidate generation and before probes. It does not use ASK control tokens.

First AskUserQuestion options:

- Proceed with the N questions as listed.
- Drop some questions.
- Add a question.
- Pivot: regenerate from a different angle.
- Skip-grill: go straight to synthesis.

If the user chooses Drop/Add/Pivot, ask one follow-up AskUserQuestion for the needed parameter:

| Choice | Follow-up |
|---|---|
| Drop some questions | Ask for IDs to drop. |
| Add a question | Ask for the question text. |
| Pivot | Ask for the new angle. |

Proceed and Skip-grill need no follow-up.

## FEYNMAN Checkpoints

Use AskUserQuestion for:

- Ambiguous session selection.
- Cold path topic/context collection.
- Failure-list resolution prompts.
- Draft review before writing `conclusion.html`.
