# Inference result contract

Version 1.1.

Every public result from an appeler name-analysis package is a DataFrame in
which each row carries, alongside the package's target-specific columns, a
common block of metadata columns defined here. The contract separates
whether a model produced a usable probability distribution from whether the
package chose to return an answer, because a model can score a name while
the package abstains when the evidence is ambiguous.

Version 1.0 shipped inside ethnicolr and assumed every estimator emits a
single predicted label. Its siblings broke that assumption in ways that were
correct, not deviant: naampy returns a calibrated score and deliberately
never names a label, and outkast returns a composition of proportions in
which no single label is the answer. Version 1.1 therefore defines three
result forms, adds two abstention reasons the packages needed, fixes the
reason spelling, and requires boolean status columns in place of bespoke
status strings.

## Result forms

Every result declares its form in the `result_form` column, constant across
the rows of one call.

`label` is the 1.0 behavior: the package returns a probability distribution
over target categories, plus `predicted_label` and `predicted_probability`.

`score` returns one calibrated probability for a stated quantity, such as
the female share among binary source labels, and no label. The
`predicted_label` and `predicted_probability` columns are omitted entirely,
not left missing: absence prevents downstream code from treating a missing
label as an unknown one and manufacturing hard classifications the package
refused to make.

`composition` returns proportions across categories that sum to one, with
optional count and support columns, as from a deterministic aggregate
lookup. `predicted_label` is omitted for the same reason: the composition is
the answer.

## Common columns

| Column | Meaning |
| --- | --- |
| `inference_contract_version` | Version of this result contract. `1.1` under this document. |
| `estimate_type` | Always `name-pattern estimate`. |
| `result_form` | `label`, `score`, or `composition`. |
| `target` | Quantity estimated, such as `race-ethnicity` or `country-origin`. |
| `input_scope` | Name components used: `first-name`, `last-name`, or `full-name`. |
| `predicted_label` | Label form only. Highest-probability returned label, or missing after abstention. |
| `predicted_probability` | Label form only. Probability of `predicted_label` on a 0 to 1 scale. |
| `scored` | Boolean. Whether the model or table produced a usable result for this row. |
| `script_supported` | Boolean. Whether the input script is supported. |
| `abstained` | Boolean. Whether the package declined to return an answer. |
| `abstention_reason` | Machine-readable reason, present exactly when `abstained` is true. |
| `model_id` | Stable identifier for the model or dictionary. |
| `model_version` | Version of the package producing the estimate. |
| `model_revision` | Immutable revision of every artifact used for inference. |
| `reference_population` | Population represented by the training or lookup data. |
| `calibration_status` | Whether and how probability calibration was validated. |
| `calibration_reference` | Data population used to assess or fit calibration. |
| `uncertainty_method` | Method used for uncertainty output, when requested. |
| `uncertainty_level` | Requested interval, range, or coverage level. |

`scored` and `abstained` are boolean columns. A package must not encode
these states in a status string of its own design; a consumer that filters
on `abstained` must get the same behavior from every package in the fleet.

If an input column uses a name reserved for estimator output, the package
preserves the input under `input_<name>`, adding a numeric suffix when that
name already exists.

## Invariants

All forms:

- An unscored row must abstain.
- `abstention_reason` is present if and only if `abstained` is true.
- An unsupported-script row cannot be scored.
- Every probability, score, or proportion is finite and between zero and
  one. Probabilities and proportions over categories sum to one, apart from
  documented rounding.
- `model_revision` identifies an immutable, complete artifact bundle. A
  mutable branch name such as `main` or `latest` is not a revision.

Label form:

- A row that does not abstain has a predicted label and probability, and
  `predicted_probability` equals the target probability associated with
  `predicted_label`.
- A scored row may still abstain under an uncertainty rule. The probability
  distribution remains available and `predicted_label` is missing.

Score form:

- A row that does not abstain has its score column populated.
- No label or per-category probability columns exist in the result.

Composition form:

- A row that does not abstain has every category proportion populated.
- Count and support columns, where present, are consistent with the
  proportions.

## Abstention reasons

The shared vocabulary is `missing-name`, `no-letters`,
`unsupported-script`, `unsupported-context`, `input-truncated`,
`out-of-vocabulary`, `out-of-dictionary`, `uncertain-score`, and
`insufficient-evidence`.

Reasons are lowercase tokens with hyphens, never underscores. Version 1.1
adds `unsupported-context`, for a row whose required context, such as a
state or year, is outside what the artifact covers, and `input-truncated`,
for a name shortened to a model's input limit before scoring. A package may
add a reason only when none of these is accurate, and should propose it for
this list in the same change.

Disclosure suppression, where a source cell exists but policy withholds it,
reports `insufficient-evidence`; the package's documentation, not the
reason column, explains the suppression policy.

## Uncertainty columns

Statistical interval columns use `<probability_column>_lower` and
`<probability_column>_upper`. Monte Carlo dropout summaries use
`<probability_column>_mc_mean`, `_mc_std`, `_mc_lower`, and `_mc_upper`,
because their empirical quantiles are not automatically confidence
intervals.

## Intended use

These outputs describe patterns associated with names in a stated reference
population. They do not establish a person's identity, ancestry,
citizenship, religion, caste, race, or ethnicity. Do not use them for
individual profiling or consequential decisions. Prefer aggregate analysis,
disclose abstention and coverage, and report sensitivity to the reference
population.

## History

1.1 (this document): moved the contract to this repository; added
`result_form` with the score and composition forms; omitted rather than
blanked the label columns outside the label form; required boolean `scored`
and `abstained`; ruled reason spelling hyphenated; added
`unsupported-context` and `input-truncated`.

1.0 (ethnicolr 2.0.0, `docs/source/inference_contract.md`): initial
contract, label form only.
