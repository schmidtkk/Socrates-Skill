# Artifacts, Probes, and Verification

This file defines the durable outputs and evidence rules shared by all Socratic commands.

## Topic Modes

| Mode | What `<topic>` is | Example |
|---|---|---|
| `concept` | A concept, definition, formula, or paper section. | `concept "diffusion guidance"` |
| `project` | A path: file, directory, or `.` for the whole repo. | `project src/auth` |

Only `concept` and `project` are valid for ASK and THINK. `paper` folds into `concept`; `task` is out of scope.

## Session Tree

ASK and THINK sessions write under:

```text
.socrates/
  <YYYY-MM-DD-HHMM>-<mode>-<topic-mode>-<topic-slug>/
    self-check.json
    candidates.json          # THINK always; ASK only if explicitly persisted later
    priming/                 # THINK project mode only
    evidence/
    transcript.md            # ASK mode
    mechanism.md             # ASK/THINK synthesis output
    conclusion.html          # FEYNMAN warm output, if run
```

Slug derivation: lowercase the topic, replace non-alphanumerics with `-`, truncate to 40 chars.

FEYNMAN warm path writes `conclusion.html` into a prior session directory. FEYNMAN cold path creates:

```text
.socrates/<YYYY-MM-DD-HHMM>-feynman-cold-<topic-slug>/conclusion.html
```

Cold path must not fabricate `mechanism.md`, `candidates.json`, or evidence.

## Probe Rules

| Probe type | Auto-run? |
|---|---|
| Read / Grep / Glob | YES |
| WebSearch / WebFetch | YES |
| Read-only Bash (`git log`, `git show`, `git diff`, `ls`, `cat`, `wc`) | YES |
| Read-only MCP (`zread`, `gitnexus query`, `context7`) | YES |
| Bash with side effects (`mv`, `rm`, `git checkout`, `git commit`, script execution) | NO, enumerate as pending probe |
| Network writes (POST/PUT/DELETE outside WebFetch GET) | NO, enumerate as pending probe |
| Code edits | NO, enumerate as pending probe |

Probe outputs go to `evidence/NN-<short-name>.{txt,json,md}` numbered sequentially.

## Verification Tags

Tag every non-trivial claim in `mechanism.md` sections 1-7:

| Tag | Meaning |
|---|---|
| `[verified: evidence/NN]` | A probe directly supports the claim. |
| `[user-asserted]` | ASK mode: user stated it and no probe verified it. |
| `[unverified]` | THINK/FEYNMAN: claim is stated but no probe established it. |
| `[assumption]` | Claude inferred it to make the model cohere. |

In narrative synthesis, inherit tags as inline markers:

| Marker | Source tag |
|---|---|
| `^v` | `[verified: evidence/NN]` |
| `^p` | `[user-asserted]` |
| `^u` | `[unverified]` |
| `^a` | `[assumption]` |

Do not collapse `^p` and `^u`: user-owned unverified claims and model-owned unverified claims have different accountability.
