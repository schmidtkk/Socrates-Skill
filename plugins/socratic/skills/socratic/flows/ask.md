# ASK Mode Flow

Command: `/socratic:ask <concept|project> <topic>`

ASK grills the user one question at a time, then synthesizes the user's answers into `mechanism.md`.

Read before running:

- `contracts/interaction.md`
- `contracts/artifacts.md`
- `taste/critic-rubric.md`
- `taste/seeds/<topic-mode>.md`
- `synthesis/mechanism.md`

## Flow

1. Parse args. Topic mode must be `concept` or `project`; otherwise show usage and stop.
2. Create `.socrates/<ts>-ask-<topic-mode>-<topic-slug>/`.
3. Opening ritual: issue one AskUserQuestion with 2-3 questions about the user's current cognitive state.
   - Always cover understanding level and mechanism gap focus.
   - In project mode also ask which files/areas the user has already looked at.
   - Persist answers to `self-check.json`.
4. Generate at least 12 candidate questions tied to the topic and ritual answers.
5. Run the critic in `taste/critic-rubric.md`; keep only questions that pass both axes.
6. Ask the highest-impact kept question using the ASK grill format from `contracts/interaction.md`.
7. Per normal answer:
   - Ask "why did you pick that option?"
   - Append `{question, answer, why, other_used}` to `transcript.md`.
   - Re-score remaining candidates quickly; drop answered/stale ones; regenerate if needed.
8. Per control token:
   - `skip`: record control event; continue.
   - `enough`: synthesize immediately.
   - `pivot`: record control event; regenerate from the new angle; continue.
9. Budget:
   - Target 3-5 grill questions.
   - Use Q6/Q7 only for a new high-value mechanism gap.
   - Q7 is the absolute cap.
10. Synthesize when budget ends, user says `enough`, or no kept candidates remain.

## Synthesis

Pass 1 fills `mechanism.md` sections 1-9 from:

- `transcript.md`
- `self-check.json`
- `evidence/*` if any probes ran

ASK claims are usually `[user-asserted]`. Use `[verified: evidence/NN]` only when an actual probe supports the claim. Use `[assumption]` for model-inferred connective tissue.

Pass 2 writes section 0 using `synthesis/mechanism.md`.

ASK section 0 must use authorship-aware framing: "your account so far is..." / "the model you traced converges on...". Do not assert user answers as truth.
