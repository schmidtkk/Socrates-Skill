# Six Quality Axes (PRD §0 Taste Layer)

A candidate question is kept ONLY if it passes all six axes. Failing any one → drop.

This is the second axis of the critic (the first being Paul/Elder type in `paul-elder-types.md`).

---

## 1. Mechanism-revealing

**Test:** Does the question's answer expose *why* something is designed/defined/structured this way, rather than *what* it is or *how* to use it?

| Pass | Fail |
|---|---|
| "Why does CFG subtract the unconditional score instead of weighting it?" | "What is the formula for CFG?" |
| "Why does this module hide the lock behind a context manager?" | "How do you call this module?" |

**Why this axis:** without it, the skill degenerates into a knowledge quiz.

---

## 2. Uncertainty-reducing

**Test:** Does the user (or Claude in think mode) currently *not know* the answer, and will answering it reduce a known unknown?

| Pass | Fail |
|---|---|
| Asks something the user marked "I'm not sure" in self-check | Asks something the user clearly stated in self-check |
| Targets an `[assumption]` claim in `mechanism.md` so far | Targets a `[verified]` claim |

**Why this axis:** without it, the skill asks questions that confirm what's already known.

---

## 3. Discriminative

**Test:** Does the question's answer distinguish between two or more competing explanations / designs / mechanisms?

| Pass | Fail |
|---|---|
| "Is the guidance term acting as a score perturbation or as a distillation target?" (two competing interpretations) | "What does guidance do?" (only one direction) |
| "Why is the sentinel file in the runtime dir vs the project dir? What does each location protect?" | "Where is the sentinel file?" |

**Why this axis:** non-discriminative questions only get one-bit answers ("yes/no", "X"), and waste budget.

---

## 4. Falsifiable

**Test:** Can the answer be verified by evidence — proof, counterexample, code path, test, derivation, experiment? If yes, is there a concrete probe attachable to this question?

| Pass | Fail |
|---|---|
| "Why is condition X necessary in the proof?" → counterexample probe | "What does X mean philosophically?" |
| "What would break if we removed this guard?" → grep callers + reason about invariants | "What's elegant about this design?" |

**Why this axis:** unfalsifiable questions produce decorative answers ("it's elegant", "it's natural"). Drop them.

---

## 5. Grounded

**Test:** Is the question bound to specific material — code path, paper section, formula, doc page, experimental result? Or is it floating in abstraction?

| Pass | Fail |
|---|---|
| "In `src/auth/session.py:42`, why does `lock_token` happen *before* `verify_signature` and not after?" | "What's the right order for auth steps?" |
| "In Ho & Salimans §3, why is the unconditional model trained with random label dropout?" | "What's the principle behind classifier-free guidance?" |

**Why this axis:** ungrounded questions get ungrounded answers, which can't be verified by probe.

---

## 6. Actionable

**Test:** Will the answer change a specific next step — the user's plan, their verification approach, their risk assessment, their understanding model, or what to write into memory?

| Pass | Fail |
|---|---|
| "If guidance is actually distillation, does this change which loss you'd use to fine-tune?" → changes the plan | "Is guidance a kind of distillation in spirit?" → changes nothing |
| "Does the lock token need to outlive the session, or just the request?" → changes invariants | "Why is locking important in general?" |

**Why this axis:** without actionability, the question is intellectually pretty but operationally useless. The skill is not a philosophy club.

---

## Rubric: when scoring a candidate

Score each axis as `true`/`false`. Record in `candidates.json`:

```json
"quality_scores": {
  "mechanism_revealing": true,
  "uncertainty_reducing": true,
  "discriminative": false,
  "falsifiable": true,
  "grounded": true,
  "actionable": true
}
```

If ANY axis is `false` → `keep_or_drop: "drop"` with a `drop_reason` field naming the failed axis.

There is no partial credit. The whole point of taste is to refuse marginal questions, not to weight them.
