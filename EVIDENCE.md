# Operation evidence record

Template for Appellation 1.2. Keep completed records in the implementing package
and link them from [ADOPTION.md](ADOPTION.md). One record covers a specified
operation, artifact, and claim scope; related operations may share sections when
their differences are explicit. This template collects evidence for
[STANDARD.md](STANDARD.md); filling it in does not establish conformance.

Use immutable links or content hashes for completed assessments. Write
`unassessed` when evidence is missing and explain every `not-applicable`. Do not
replace existing model cards or data dictionaries with duplicated prose: link to
their versioned sections.

## Identity and scope

| Field                          | Record                                                                                    |
| ------------------------------ | ----------------------------------------------------------------------------------------- |
| Operation and options assessed | Public function, path through any hybrid, result form, and option values                  |
| Basis                          | Lookup, learned model, or derived calculation                                             |
| Standard and contract          | Version 1.2 and the Appellation commit or release used                                    |
| Implementation                 | Package version and code commit                                                           |
| Artifacts                      | Bundle and component revisions or content hashes, including preprocessing and calibration |
| Evidence                       | This record's revision, assessment date, reviewer, and linked evaluation-code revision    |
| Claim scope                    | Exact source population, intended application population, and supported claim             |

Keep specification, implementation, artifact, and evidence versions separate. A
revised report need not change weights; new weights require a new assessment.
Fingerprints must identify stable content without requiring a manifest and
report to contain each other's final hashes.

For inherited requirements, also record the applicable py-canon revision and the
checker versions used. A moving workflow tag names an update channel; record its
resolved commit for a reproducible assessment.

## Data and target

Address D01 and D02:

- Define one source observation and one input row. Record collection dates,
  geography, label source, duplicate handling, and source revisions.
- Give the target definition or formula, numerator and denominator, categories,
  conditioning variables, and evaluation weights.
- Record input normalization, supported scripts and lengths, selection, excluded
  labels, retained-cell thresholds, and suppression. Explain how these affect
  the denominator and coverage.
- Link the data dictionary, recodes, join checks, and source audits. For a
  derived result, identify each component and its population and dates.

## Evaluation and uncertainty

For a lookup, document table reconciliation, support and suppression checks, and
any sampling design used to infer beyond observed counts. For a model, document
partitions, grouping, leakage checks, baselines, selection and calibration
procedures, and untouched evaluation results. For a derived calculation, give
the formula, assumptions, component dependence, and any independent validation
of its interpretation.

For each claimed quantity, record:

| Item                | Required content                                                                                                        |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Acceptance criteria | Criteria, rationale, and when they were chosen relative to final evaluation                                             |
| Performance         | Metrics with units, weights, denominators, intervals, baselines, and exact artifact identity                            |
| Calibration         | Fitting versus assessment populations, method or no adjustment, diagnostics and results, or justified non-applicability |
| Coverage            | Input eligibility, abstention by reason, answered-row performance, and relevant strata                                  |
| Uncertainty         | Target, method, nominal level, assumptions, independent/resampling unit, and validation of any coverage claim           |
| Omissions           | Unquantified sources of error; limits of per-name, aggregate, or model-variability summaries                            |
| Transfer            | Population differences, independent aggregate validation where available, and sensitivity checks; otherwise unassessed  |
| Reproduction        | Commands, environment, data/split/code hashes, and access restrictions                                                  |

An interval on model performance cannot fill the per-name uncertainty entry. A
derived target generated by the same formula cannot establish validity against
an independently observed outcome.

## Assessment

Use one row per operation and scope, expanding these rows where necessary. For
output and artifact checks use `pass`, `fail`, or `unassessed`. For empirical
claims use `supported`, `unsupported`, or `unassessed`. `not-applicable`
requires a reason tied to the operation. Support is always limited to the stated
claim and population; it is not universal approval.

| Assessment                    | Status     | Scope, evidence, and outstanding work                                                               |
| ----------------------------- | ---------- | --------------------------------------------------------------------------------------------------- |
| Output behavior               | unassessed | Link actual contract/API tests and local results; include relevant CONFORMANCE.md cases             |
| Artifact integrity            | unassessed | Link manifest, hash/schema checks, installed-artifact tests, and disclosure checks where applicable |
| Source-population claims      | unassessed | Link reconciliation or held-out model/derived-result evidence                                       |
| Calibration claim             | unassessed | Link diagnostics and acceptance decision, or justify non-applicability                              |
| Uncertainty claim             | unassessed | Identify the specific quantity and supported interpretation                                         |
| Application-population claims | unassessed | Link transfer/aggregate evidence separately from source-population evidence                         |

Record the reviewer's decision and unresolved limitations. Changes to targets,
data, artifacts, transformations, or decision rules require reassessment of
affected rows. An application with no transfer evidence must remain unassessed
even when all mechanical checks pass.

Record which checks ran and which were skipped, unavailable, or inapplicable,
with reasons. No executed checks is not a pass. A waived failure retains its
status and the reviewer's rationale; do not relabel it as supported or
not-applicable.
