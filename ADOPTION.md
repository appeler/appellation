# Adoption

Status as of 2026-08-19, from reading each repository at its branch tip.
Version states name what is on PyPI versus what is built locally, because
two packages have unpublished breaking releases that migration should fold
into.

## Status

| Standard | ethnicolr | naampy | pranaam | outkast | instate |
| --- | --- | --- | --- | --- | --- |
| Verbs (`lookup_*` / `estimate_*`) | yes | yes | yes | yes | yes |
| Canonical signature (`data`, `*_column`, keyword-only) | yes | no, sequence-only | no, sequence-only | close, arg named `frame` | yes |
| 0 to 1 calibrated probabilities | yes | yes | yes | yes, proportions | yes, temperature-scaled |
| Explicit abstention, shared reasons | yes | yes, hyphenated | yes, hyphenated | yes, own `secc_*` names, underscores | yes, hyphenated |
| Contract columns | 1.0 in full | parallel names, gaps | right columns, hand-built inline | own vocabulary | 1.1 in full, composition form |
| Uncertainty mechanism | conformal, MC, Wilson, prior | none in API | none, fixed 0.8 threshold | none, enumeration | calibration only; intervals open |
| Artifacts on HF, SHA-pinned | yes | yes, plus per-file SHA-256 | yes, plus per-file SHA-256 | vendored, 628 KB, hash in source | yes, plus per-file SHA-256 |
| Runtime Parquet and JSON, no CSV | yes | yes | yes, no tables | yes | yes |
| Non-identity framing | yes | yes | yes | yes | yes |
| Evaluation contract met by shipped artifacts | yes | yes | yes | not applicable | yes |

Instate's column reads its 3.0 branch
([appeler/instate#53](https://github.com/appeler/instate/pull/53)), the
first migration completed under this standard and the first emitter of
contract 1.1. Its language target also moved from invented geometric
weights to Census 2011 C-16 mother-tongue shares, and a recorded experiment
justifies deriving language from the state composition instead of training
a second model (held-out log loss 1.523 for the transform vs 1.566 direct).

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

### instate (3.0.0 in review, PyPI has 2.0.0)

Migrated ([#53](https://github.com/appeler/instate/pull/53)): composition
API under contract 1.1, retrained and temperature-scaled model with
untouched-test metrics (modal top-1 0.534 / top-3 0.770), census-based
language shares replacing the geometric weights, artifacts on the pinned
Hub revision with per-file SHA-256, wheel down from 34.5 MB to 50 KB. The
2.1.0 build was skipped; everything ships as 3.0.0. Remaining after merge:
an interval mechanism to meet the uncertainty bar, and release to PyPI.

## Sequencing

Outkast first, because its unpublished 2.0.0 is a closing window. Then
naampy and pranaam, each a small breaking release. ethnicolr's own 1.1
bump can ride any of these. Instate last and started early, because
retraining is the only long-running work. The shared conformance test
suite is worth building after the second conforming package, when the
duplication is real rather than predicted.
