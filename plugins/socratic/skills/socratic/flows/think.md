# THINK Mode Flow

Command: `/socratic:think <concept|project> <topic>`

THINK self-grills with one mandatory user review checkpoint, runs read-only probes, then writes `mechanism.md`.

Read before running:

- `contracts/interaction.md`
- `contracts/artifacts.md`
- `taste/critic-rubric.md`
- `taste/seeds/<topic-mode>.md`
- `synthesis/mechanism.md`

## Startup

For `concept`: think-then-fetch.

1. Generate candidate questions from the topic and Claude's current knowledge.
2. Verify later during probes, usually with WebSearch/WebFetch against authoritative sources.
3. No `priming/` directory is written unless a later probe creates evidence.

For `project`: fetch-then-think.

1. Write priming files under `priming/`.
2. Read `README.md`, package metadata if present, and `CLAUDE.md` if present.
3. If `CLAUDE.md` is gitignored, record that in `priming/00-sources.md` as local-only context.
4. Glob top-level structure, max 100 entries.
5. If topic is a path, read up to 5 main files (`*.py`, `*.ts`, `*.js`, `*.go`, `*.md`).
6. Grep for likely entry symbols (`main`, `__init__`, `default export`, route registrations).
7. Generate candidate questions grounded in the priming material.

## Candidate Generation

1. Fill `self-check.json` with `"mode": "think"` and `"virtual": true`.
2. Generate at least 8 candidates; aim for 10-15.
3. Run `taste/critic-rubric.md`.
4. Write all candidates to `candidates.json`.

Candidate entries must include:

```json
{
  "id": "Q1",
  "question": "...",
  "paul_type": "evidence | assumption | clarification | perspective | consequence | meta",
  "target_uncertainty": "...",
  "why_it_matters": "...",
  "expected_answer_type": "definition | evidence | counterexample | constraint | test | derivation",
  "decision_impact": "understanding | plan | risk | verification | memory",
  "probe": "<concrete read-only probe action>",
  "quality_scores": {
    "mechanism_revealing": true,
    "uncertainty_reducing": true,
    "discriminative": true,
    "falsifiable": true,
    "grounded": true,
    "actionable": true
  },
  "keep_or_drop": "keep | drop",
  "drop_reason": "...",
  "final_status": "kept | dropped_by_critic | dropped_by_user | added_by_user"
}
```

`keep_or_drop` is the critic's frozen verdict. `final_status` is the canonical downstream filter.

Initial mapping:

| Critic verdict | `keep_or_drop` | `final_status` |
|---|---|---|
| survives both axes | `keep` | `kept` |
| fails any axis | `drop` | `dropped_by_critic` |

## Review Checkpoint

Print the kept list as a compact markdown table, plus 2-3 representative dropped candidates with reasons.

Then follow the THINK review checkpoint in `contracts/interaction.md`:

- Proceed: continue unchanged.
- Drop: ask for IDs, set each to `dropped_by_user`.
- Add: ask for text, append `Q-user-N`, fast-critic it, ask confirmation if it fails, then derive a read-only probe.
- Pivot: ask for angle, regenerate, allow at most one pivot.
- Skip-grill: set kept entries to `dropped_by_user`, skip probes, synthesize with low rigor.

Update `candidates.json` after the checkpoint.

## Probes

For each entry with `final_status` `kept` or `added_by_user`, run the probe subject to `contracts/artifacts.md`, write evidence, and attach the evidence reference to the candidate.

## Synthesis

Pass 1 fills sections 1-9 from:

- `candidates.json`
- `evidence/*`
- `priming/*` for project mode
- `self-check.json`

Tag claims as `[verified: evidence/NN]`, `[unverified]`, or `[assumption]`. Sections without verifiable content get `N/A: <reason>`.

Pass 2 writes section 0 using `synthesis/mechanism.md`. THINK mode may assert directly, but must preserve verification markers and gap honesty.
