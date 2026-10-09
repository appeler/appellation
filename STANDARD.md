# The appeler standard

Version 1.2.

This standard defines what evidence must accompany a name-analysis operation.
[CONTRACT.md](CONTRACT.md) defines its returned rows. A package may satisfy that
output contract while lacking evidence for calibration, uncertainty, or use in
another population. These assessments must remain separate.

The requirements below are normative. `Must` is required; `should` permits a
documented, evidence-based exception; `may` is optional. Identifiers refer to
requirements within this version. Examples and tools help apply the requirements
but do not replace them.

## Target and reference data

**D01:** Each public operation must have an [evidence record](EVIDENCE.md)
linked from its documentation and identified by the artifact manifest. Define
the quantity being estimated before choosing a model or uncertainty method. The
record must specify:

- Input unit, required context, normalization, supported scripts and lengths,
  and any transformations that lose information.
- Source and label provenance, geography, collection period, observation unit,
  duplicate handling, selection, exclusions, and disclosure limits.
- Target categories, numerator, denominator, conditioning variables, and
  weighting. Distinguish source records from unique people and observed source
  labels from personal identity.
- Intended application population and the evidence supporting use there. Missing
  evidence must be recorded as unassessed.

For example, a female-label share among retained female and male electoral
records has a different denominator from a share among all source labels or all
residents. Renormalizing retained categories or suppressing cells changes the
quantity and must be disclosed. A broad target name such as `race-ethnicity` is
only a key into this definition.

## Requirements by operation

**D02:** Assign each operation a basis below, independently of its result form
(`label`, `score`, or `composition`). A hybrid must identify the basis used for
each row and satisfy the requirements of each path.

| Basis               | What the result represents                           | Required evidence                                                                                                                                |
| ------------------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Lookup              | A descriptive quantity in a specified table          | Source and denominator audit, count/share reconciliation, coverage, and suppression checks; sampling inference only when justified               |
| Learned model       | An estimate from fitted parameters                   | Separate development and evaluation data, probabilistic performance, calibration assessment, and uncertainty appropriate to the claimed quantity |
| Derived calculation | A transformation or combination of source quantities | Formula, all source revisions, assumptions, aligned categories and populations, and evaluation of any claim beyond the calculation itself        |

An exact lookup needs no train/test split to establish that it reproduces its
table. Its use to estimate another population does require evidence. A
composition produced by a model still needs model evaluation.

## Verbs and signatures

**A01:** Public name-analysis functions use `lookup_*` for a table lookup and
`estimate_*` for a model or derived calculation. A lookup must not silently fall
back to a model. Metadata utilities may use descriptive verbs such as `list_*`.

The canonical signature takes a DataFrame and an input-column name, with options
keyword-only:

```python
estimate_target(data: pd.DataFrame, surname_column: str, *, ...) -> pd.DataFrame
```

Column arguments are `surname_column`, `first_name_column`, or `name_column`, as
appropriate. Required contextual columns or scalar options must be documented.
The frame argument is `data`.

**A02:** Return a copy with result columns appended; preserve row count, order,
and index, including duplicate indices and empty inputs. Do not mutate the
input. Handle reserved-column collisions as specified in the contract. A
single-name-column operation may additionally accept a string, sequence, or
Series with identical result semantics.

**A03:** Invalid calls raise; unsupported observations abstain. Missing or
duplicate columns and invalid option types raise immediately. A name or context
outside the artifact's documented domain yields an abstention reason. Preserve
that row with missing estimates.

## Probabilities, calibration, and abstention

**P01:** Probabilities and proportions use the 0 to 1 scale. A model must not
return a default distribution for an unsupported input. An unseen name is not
necessarily unsupported: a model may generalize to unseen names within an
evaluated input domain. A lookup miss must abstain.

**P02:** Before release, a learned probability estimate must pass a documented
calibration assessment on held-out data under criteria chosen before inspecting
final test results. Report the assessment population, unit, weighting,
reliability diagnostics, log loss or Brier score, and uncertainty at the
independent unit. Report relevant strata and unsupported or sparse strata. No
universal error threshold applies to every task; the evidence record must state
and justify the acceptance criteria.

Post-hoc calibration is optional when the unadjusted model meets those criteria.
When fitted, report its fitting data and compare adjusted and unadjusted
probabilities on the same untouched evaluation set. Applying temperature or
Platt scaling alone is not validation. These methods are procedures whose
performance must be measured
([Guo et al., 2017](https://proceedings.mlr.press/v70/guo17a.html)).

`calibration_status` must distinguish an assessment that passed its stated
criteria from one that failed or was not performed, and identify any adjustment
method. Package documentation must define the values; the evidence record
supplies the criteria and results. A direct descriptive lookup reports
`not-applicable`. A derived probability claim needs its own assessment;
component calibration does not establish calibration of the derived output. A
derived descriptive mixture may report `not-applicable` only when explicitly
limited to that calculation.

Models that fail or lack the required assessment must not expose their scores as
validated probabilities in a conforming release. Developmental results must be
identified as such and cannot receive an empirical-support pass.

**P03:** An ambiguous supported score or composition remains a valid answer. Do
not require a classification threshold for an operation that does not return a
label. Where a decision rule abstains, expose its threshold as a keyword option
and report performance and coverage under that rule. Never present performance
among answered rows as performance on all inputs.

## Uncertainty

**U01:** Every operation must state the quantity its uncertainty concerns, the
sources of variation it covers, and those it omits. Use the following
requirements instead of requiring every package to offer one method from a
common menu.

| Quantity or claim                                                 | Requirement                                                                                                                                                                                                    |
| ----------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Descriptive proportion in a fixed enumeration or released extract | Report denominator, coverage, filtering, rounding, and suppression. Sampling intervals are not required.                                                                                                       |
| Population share estimated from a sample                          | Use intervals appropriate to the sampling design, weights, clustering, and denominator. A binomial interval requires its own sampling assumptions.                                                             |
| Learned score or composition                                      | Report uncertainty in held-out performance and calibration. Per-name intervals are required only when the operation claims to quantify per-name estimation uncertainty, and must be validated for that target. |
| Prediction set                                                    | Specify the predicted outcome and nominal coverage, the assumptions for coverage, and achieved coverage and set size on independent evaluation data.                                                           |
| Derived quantity or aggregate application                         | Propagate relevant component uncertainty and dependence, or state what is held fixed; assess sensitivity to structural assumptions and population mismatch.                                                    |

A census enumeration can contain coverage and recording errors without having
sampling error for its observed totals. Interpreting its counts as a sample from
a broader population requires an explicit additional model. Wilson bounds do not
measure these other errors. The Census surname files are enumeration aggregates
with disclosure suppression
([Census Bureau](https://www.census.gov/data/developers/data-sets/surnames.html)).

**U02:** A bootstrap interval for overall log loss or accuracy must stay with
that metric. It is not a per-name probability interval. Ensemble or Monte Carlo
dropout variation may be reported as model variability, but does not establish
confidence-interval coverage. Declare the stochastic procedure and validate any
stronger claim. Conformal sets usually offer marginal coverage under
exchangeability, not automatic coverage for each name, subgroup, or shifted
population ([Angelopoulos and Bates](https://arxiv.org/abs/2107.07511)).

Use the contract's uncertainty column names and record the method actually
returned. When no defensible per-name interval is available, say so and omit its
endpoint columns; do not invent an interval to satisfy conformance.

## Evaluation and transfer

**E01:** Split at the unit required by the claim, before augmentation,
balancing, or learned preprocessing. For unseen-name performance, keep
normalized, representation-equivalent names in one partition. Account for
repeated people, households, aliases, and shared source records. For a claim
about new regions, sources, or periods, evaluate those held-out contexts. Random
row splitting alone does not establish either claim.

Keep training, model selection, calibration fitting, and final evaluation
separate. A cross-fitting alternative must document how each assessed
observation is excluded from every fitted component used to predict it. Learn
vocabularies on training data, or identify externally fixed vocabularies and any
exposure to evaluation sources. A conformal procedure must justify its
calibration design separately from probability calibration.

**E02:** Compare models and relevant simple baselines on the same clean
evaluation data. Report name-weighted and record-weighted metrics where counts
permit; call weights population weights only when justified. Publish coverage,
abstention reasons, performance on answered rows, and diagnostics by relevant
script, source, geography, support, and name novelty. Resampling must respect
the independent unit. Mark contaminated or reused evaluation data as
developmental.

**E03:** Calibration and uncertainty evidence apply to the evaluated population.
Transfer to another population must be assessed, or explicitly marked
unassessed. Changes in source, time, selection, or geography can invalidate
uncertainty estimates
([Ovadia et al., 2019](https://proceedings.neurips.cc/paper_files/paper/2019/hash/8558cb408c1d76621371888657d2eb1d-Abstract.html)).

A prior-shift option is optional. It must state the assumed stability of the
input distribution within each class, the source and target priors, support
conditions, and sensitivity to violations. Prior adjustment does not correct
arbitrary population differences and must preserve unsupported-input
abstentions. Reapplying a label decision rule to supported adjusted
distributions follows C06.

**E04:** Validate aggregate applications against independently observed
aggregate quantities with matched definitions where available. Report weighting,
selection from abstention, and sensitivity to reference populations and missing
inputs. Without such evidence, aggregate validity is unassessed. Averaging
scores or banning individual classification does not itself validate an
aggregate estimator.

**E05:** Derived calculations must distinguish an exact arithmetic result from
its substantive interpretation. Mixing surname-specific state shares with
statewide language shares defines a mixture. Interpreting it as a surname's
language distribution additionally assumes language is independent of surname
conditional on state, with compatible populations and dates, or requires
evidence for a suitable alternative. Evaluation against targets generated by the
same mixture does not validate that assumption against observed language data.

## Artifacts and reproducibility

**R01:** Runtime tables are typed Parquet validated against an explicit Arrow
schema at load. Vocabularies, labels, calibration statistics, and manifests are
schema-versioned JSON. CSV may be used for transport, acquisition, and training;
it is not a runtime table format. These requirements apply the Python fleet's
[runtime-asset baseline](https://github.com/gojiplus/py-canon/blob/main/STANDARD.md#runtime-assets)
to name-analysis tables and metadata; record the baseline revision used in the
assessment.

Learned model weights and serialized estimators live outside the wheel,
regardless of size, as py-canon requires. Host them in the `gojiberries` Hugging
Face organization at a full 40-character commit SHA with a model card. Lookup
tables and non-weight metadata may ship in a wheel when their bundle is at or
below 5 MB; larger bundles use the same pinned hosting with a data card. The
size allowance never exempts learned weights. These are engineering conventions,
not evidence of statistical quality. Both paths require a trusted manifest and
per-file SHA-256 verification before use. A hash manifest downloaded alongside
files must itself be anchored to a trusted revision or digest.

Packages must support a local mirror through `<PACKAGE>_MODEL_DIR` for model
bundles or a documented analogous environment option for lookup bundles. Verify
mirrors under the same rules and identify their origin and content digest in
output provenance.

**R02:** Preserve hashes or immutable revisions for source data, split
membership, preprocessing and evaluation code, fitted artifacts, and the
evidence record. The record must assess the exact shipped bundle. An older
artifact may be evaluated on genuinely untouched data; retraining is required
when needed to remedy leakage or inadequate performance, not merely because the
evaluation document is newer. Restricted source data may remain restricted;
disclose limits to independent reproduction.

**R03:** Sensitive aggregate releases must document suppression and test whether
published totals or overlapping tables reconstruct withheld cells. A minimum
cell count alone is not a disclosure guarantee. Distinguish reproducible private
training inputs from artifacts approved for release.

## Framing

Documentation and examples must describe name patterns or observed group
compositions under the stated reference population. They must not present
outputs as establishing a person's identity, ancestry, citizenship, religion,
caste, race, ethnicity, gender, residence, or language. Individual profiling and
consequential decisions are outside the intended use. Function and column names
must name the estimated quantity. Aggregate research still requires the
validation described above.

## Conformance and maintenance

Record conformance per operation and artifact using [EVIDENCE.md](EVIDENCE.md).
Assess output behavior, artifact integrity, and empirical support separately,
with links to executable tests and evaluation results.
[ADOPTION.md](ADOPTION.md) indexes these records. Unreviewed evidence is
unassessed, not a pass; `not-applicable` needs an operation-specific reason.
Every automated assessment must identify the checker version, specification
revision, checks executed, and checks skipped or unavailable. An empty or
incomplete run cannot establish full conformance. A waiver records an accepted
limitation; it does not turn a failed or unassessed requirement into a pass.

[CONFORMANCE.md](CONFORMANCE.md) supplies boundary examples for package tests.
It is not a validator. A future shared test suite may check stable requirements
across implementations; it cannot certify statistical adequacy. Packages own
their implementation and evidence. Skills can guide this work but must reference
a specific standard version and cannot serve as conformance evidence.

Reuse [preen](https://github.com/gojiplus/preen) for the Python package checks
it already implements. Any future Appellation checker should add result behavior
and evidence-reference checks, without duplicating packaging or release logic.
Keep detection separate from fixes. A tool must not change a denominator,
evaluation split, or uncertainty interpretation merely to make a check pass.

[py-canon](https://github.com/gojiplus/py-canon) owns build, lint, type
checking, test infrastructure, documentation builds, and releases. Appellation
owns result meaning and its evidentiary requirements. Changes to this standard
go through a PR with rationale, affected operations, examples, and migration
consequences. Record semantic changes in the contract history and assess them
before declaring a new version adopted.
