# Inference result contract

Version 1.2.

Every public name-analysis result from an appeler package is a DataFrame in
which each row carries, alongside the package's target-specific columns, a
common block of metadata columns defined here. The contract separates whether a
model produced a usable probability distribution from whether the package chose
to return an answer, because a model can score a name while the package abstains
when the evidence is ambiguous.

Conformance to this contract establishes output behavior. Statistical support
for an operation is assessed separately under [STANDARD.md](STANDARD.md) using
an [evidence record](EVIDENCE.md). The version column does not certify
calibration, population transfer, or suitability for a research question.
Metadata utilities, such as listing supported states, are outside this result
contract.

## Result forms

Every result declares its form in the `result_form` column, constant across the
rows of one call.

`label` is the 1.0 behavior: the package returns a probability distribution over
target categories, plus `predicted_label` and `predicted_probability`.

`score` returns one probability estimate for a stated quantity, such as the
female share among binary source labels, and no label. The `predicted_label` and
`predicted_probability` columns are omitted entirely, not left missing: absence
prevents downstream code from treating a missing label as an unknown one and
manufacturing hard classifications the package refused to make.

`composition` returns proportions across mutually exclusive, exhaustive
categories within the declared denominator. These sum to one and may have count
and support columns. Both `predicted_label` and `predicted_probability` are
omitted: the composition is the answer.

Result form describes the output shape, independently of whether it comes from a
lookup, a learned model, or a derived calculation. Each operation's evidence
record identifies its basis. A hybrid operation must identify the basis used on
each row in a documented output column.

## Common columns

| Column                       | Meaning                                                                                                                                                          |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `inference_contract_version` | Version of this result contract. `1.2` under this document.                                                                                                      |
| `estimate_type`              | `observed name composition` for a direct lookup; `name-pattern estimate` for a learned or derived result.                                                        |
| `result_form`                | `label`, `score`, or `composition`.                                                                                                                              |
| `target`                     | Target key resolving to an exact quantity, label definition, and denominator in the artifact's evidence record.                                                  |
| `input_scope`                | Name components used: `first-name`, `last-name`, or `full-name`.                                                                                                 |
| `predicted_label`            | Label form only. Highest-probability returned label, or missing after abstention.                                                                                |
| `predicted_probability`      | Label form only. Probability of `predicted_label` on a 0 to 1 scale.                                                                                             |
| `scored`                     | Boolean. Whether the model or table produced a usable result for this row.                                                                                       |
| `script_supported`           | Boolean. Whether the input script is supported.                                                                                                                  |
| `abstained`                  | Boolean. Whether the package declined to return an answer.                                                                                                       |
| `abstention_reason`          | Machine-readable reason, present exactly when `abstained` is true.                                                                                               |
| `model_id`                   | Stable identifier for the model or dictionary.                                                                                                                   |
| `model_version`              | Version of the package producing the estimate.                                                                                                                   |
| `model_revision`             | Immutable revision of every artifact used for inference.                                                                                                         |
| `reference_population`       | Source population, including geography and period, resolving to the artifact's evidence record.                                                                  |
| `calibration_status`         | Assessment status and method, if any, as defined under STANDARD.md; direct descriptive lookups use `not-applicable`.                                             |
| `calibration_reference`      | Evaluation population and weighting; the evidence record separately identifies any fitting population. Missing when calibration is not applicable or unassessed. |
| `uncertainty_method`         | Method actually used for this row's uncertainty output; missing when none is returned.                                                                           |
| `uncertainty_level`          | Nominal level for that method, strictly between 0 and 1; missing when no level applies.                                                                          |

All common columns are required, except the two label-only columns. The
uncertainty columns remain present and missing when no uncertainty is returned.
Empty results carry the same schema as populated results.

`scored`, `abstained`, and `script_supported` are non-null boolean columns. For
a missing or letter-free input, `script_supported` is false and the reason
remains `missing-name` or `no-letters`. A package must not encode these states
in a status string of its own design; a consumer that filters on `abstained`
must get the same behavior from every package in the fleet.

If an input column uses a name reserved for estimator output, the package
preserves the input under `input_<name>`, adding a numeric suffix when that name
already exists.

## Invariants

The requirement identifiers below are stable references for tests and evidence
records. Their meaning is always read at a specified contract version.

All forms:

- **C01:** An unscored row must abstain, with all target probabilities, scores,
  proportions, and uncertainty summaries missing. It must not receive a default
  distribution. Disclosable support metadata may remain.
- **C02:** `abstention_reason` is a nonempty reason token if and only if
  `abstained` is true; otherwise it is missing.
- **C03:** An unsupported-script row cannot be scored.
- **C04:** Every populated probability, score, or proportion is finite and
  between zero and one. A scored row has a complete target score or
  distribution. Category distributions sum to one within a documented numeric
  tolerance. Missing estimates on unscored rows are valid.
- **C05:** `model_revision` identifies an immutable, complete artifact bundle,
  including preprocessing, calibration, and any source tables used in a derived
  result. Mutable branches and local directory paths alone are not revisions. A
  local bundle uses a content digest and declares its local origin in documented
  provenance metadata.
- **C06:** Postprocessing, including prior adjustment, preserves unscored rows,
  their missing estimates, and their abstention reasons. It must never convert
  missing values into probabilities. A documented label decision rule may be
  reapplied to a supported adjusted distribution, with a corresponding update to
  the label, probability, and abstention fields. Any adjustment and resulting
  decision rule require stated assumptions and evaluation.

Label form:

- **C07:** A row that does not abstain has a highest-probability label and its
  probability; `predicted_probability` equals the target probability associated
  with `predicted_label`. Document tie handling.
- **C08:** A scored row may still abstain from returning a label under a
  decision rule. The probability distribution remains available; both
  `predicted_label` and `predicted_probability` are missing.

Score form:

- **C09:** A row that does not abstain has its score column populated. Neither
  label columns nor a redundant per-category distribution exists. A score near
  0.5 is a valid result, not by itself grounds for abstention. For this form,
  `abstained` is exactly the inverse of `scored`.

Composition form:

- **C10:** A row that does not abstain has every category proportion populated.
  Count and support columns, where present, are consistent with the declared
  denominator and proportions. Support counts accompanying a model or derived
  estimate must be distinguished from counts that determine its proportions. For
  this form, `abstained` is exactly the inverse of `scored`.

## Abstention reasons

The shared vocabulary is `missing-name`, `no-letters`, `unsupported-script`,
`unsupported-context`, `input-truncated`, `out-of-vocabulary`,
`out-of-dictionary`, `uncertain-score`, and `insufficient-evidence`.

Reasons are lowercase tokens with hyphens, never underscores. Version 1.1 adds
`unsupported-context`, for a row whose required context, such as a state or
year, is outside what the artifact covers, and `input-truncated`, for an input
that exceeds the supported length and would require lossy truncation to score. A
package may add a reason only when none of these is accurate, and should propose
it for this list in the same change.

Disclosure suppression, where a source cell exists but policy withholds it,
reports `insufficient-evidence`; the package's documentation, not the reason
column, explains the suppression policy.

## Uncertainty columns

Statistical interval columns use `<probability_column>_lower` and
`<probability_column>_upper`. Monte Carlo dropout summaries use
`<probability_column>_mc_mean`, `_mc_std`, `_mc_lower`, and `_mc_upper`, because
their empirical quantiles are not automatically confidence intervals.

**C11:** Every uncertainty output identifies its target, interpretation,
assumptions, and validation in the evidence record. Interval endpoints are
either both missing or both finite and ordered. Bounds for probabilities lie
between zero and one. Monte Carlo standard deviations are nonnegative. Missing
target estimates have missing uncertainty summaries. An interval for an
aggregate performance metric belongs with that metric, never in a per-name
interval column. A nominal level is not evidence of achieved coverage.
Set-valued outputs must document their encoding and target.

## Intended use

These outputs describe patterns associated with names in a stated reference
population. They do not establish a person's identity, ancestry, citizenship,
religion, caste, race, ethnicity, gender, residence, or language. Do not use
them for individual profiling or consequential decisions. Aggregate applications
require validation for their target population, disclosure of abstention and
coverage, and sensitivity to the reference population.

## History

1.2 (2026-10-09): separated output conformance from empirical support; defined
lookup and estimate provenance, missing-value and postprocessing rules, and
uncertainty interpretation; clarified composition and label abstention. The
common column names are unchanged; their revised meanings require adoption
before emitting version 1.2.

1.1: moved the contract to this repository; added `result_form` with the score
and composition forms; omitted rather than blanked the label columns outside the
label form; required boolean `scored` and `abstained`; ruled reason spelling
hyphenated; added `unsupported-context` and `input-truncated`.

1.0 (ethnicolr 2.0.0, `docs/source/inference_contract.md`): initial contract,
label form only.
