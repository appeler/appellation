# Adoption

Status as of 2026-08-19, from reading each repository at its branch tip.
Version states name what is on PyPI versus what is built locally, because
two packages have unpublished breaking releases that migration should fold
into.

## Status

| Standard | ethnicolr | naampy | pranaam | outkast | instate |
| --- | --- | --- | --- | --- | --- |
| Verbs (`lookup_*` / `estimate_*`) | yes | yes | yes | yes | no |
| Canonical signature (`data`, `*_column`, keyword-only) | yes | no, sequence-only | no, sequence-only | close, arg named `frame` | partial |
| 0 to 1 calibrated probabilities | yes | yes | yes | yes, proportions | no probabilities at all |
| Explicit abstention, shared reasons | yes | yes, hyphenated | yes, hyphenated | yes, own `secc_*` names, underscores | lookups silently NaN |
| Contract columns | 1.0 in full | parallel names, gaps | right columns, hand-built inline | own vocabulary | absent |
| Uncertainty mechanism | conformal, MC, Wilson, prior | none in API | none, fixed 0.8 threshold | none, enumeration | none |
| Artifacts on HF, SHA-pinned | yes | yes, plus per-file SHA-256 | yes, plus per-file SHA-256 | vendored, 628 KB, hash in source | models yes, 35 MB tables in wheel |
| Runtime Parquet and JSON, no CSV | yes | yes | yes, no tables | yes | yes |
| Non-identity framing | yes | yes | yes | yes | yes |
| Evaluation contract met by shipped artifacts | yes | yes | yes | not applicable | no, checkpoints predate it |

## Migration items

### ethnicolr (2.0.0 on PyPI)

Emit contract 1.1: add `result_form`, bump
`inference_contract_version`. Replace `docs/source/inference_contract.md`
with a pointer to this repository. Add per-file SHA-256 verification to
`model_artifacts.py`, which currently trusts the revision pin alone.

### outkast (2.0.0 built, PyPI has 1.0.0)

Fold into the unpublished 2.0.0 so users take one break, not two: rename
the `secc_*` status columns to the contract's boolean `scored` and
`abstained` plus `abstention_reason`; respell reasons with hyphens, with
`insufficient_support` becoming `insufficient-evidence` and
`unsupported_context` becoming `unsupported-context`; add the common
columns as a `composition`-form result; rename the `frame` argument to
`data`. Delete the dead legacy `outkast/utils.py`, which is still autodoc'd.
Then publish.

### naampy (0.11.0 on PyPI)

Rename the parallel columns to contract names, `score_target` to `target`
and `calibration_population` to `calibration_reference`, and add the
missing common columns as a `score`-form result, which 1.1 now defines so
that the no-label design is conformant rather than a violation. Add the
DataFrame-and-column call form. Surface the bootstrap intervals already
computed for the model card as `_lower` and `_upper` columns. Decide
deliberately whether the dropped state and birth-year conditioning returns;
the standard does not force it, but the silence should end.

### pranaam (0.8.0 on PyPI)

Adopt the canonical DataFrame signature with keyword-only options. Replace
the hand-built inline contract block with 1.1 columns, including
`result_form` and `inference_contract_version`. Make the abstention
threshold a keyword option instead of the fixed 0.8, and add a prior-shift
option, which matters most for a binary estimator. Delete the dead v1/v2
`pranaam/model.py` and its tests, which prop up the coverage floor. Fix the
copier answer that still reads "Predict religion from names."

### instate (2.1.0 built, PyPI has 2.0.0)

The long pole, and mostly science rather than renames. Calibrate, and
likely retrain under the package's own evaluation contract, so it can
expose probabilities at all; the shipped checkpoints predate that contract
and have no publishable metrics. Rename `get_state_distribution` to
`lookup_*` and `predict_state` and `predict_language` to `estimate_*`, with
the canonical signature. Make lookup abstention explicit instead of NaN
rows. Move the 26.6 MB and 8.2 MB Parquet tables to the already-pinned
`gojiberries/instate` revision; the resolver in `_resources.py` needs no
redesign. Decide whether 2.1.0 ships first as-is or waits for the
contract release.

## Sequencing

Outkast first, because its unpublished 2.0.0 is a closing window. Then
naampy and pranaam, each a small breaking release. ethnicolr's own 1.1
bump can ride any of these. Instate last and started early, because
retraining is the only long-running work. The shared conformance test
suite is worth building after the second conforming package, when the
duplication is real rather than predicted.
