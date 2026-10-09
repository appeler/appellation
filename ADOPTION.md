# Adoption

Standard revision: 1.2, dated 2026-10-09. No package has been reassessed against
1.2 in this repository. The migration facts below were recorded on 2026-08-19;
they are historical, not a check of current releases.

## Assessment register

Conformance is assessed per operation, artifact, and claim scope. Populate this
register from package-owned [evidence records](EVIDENCE.md), with immutable
links to implementation tests, artifacts, and evaluations. Split a package row
into its individual operations when assessing it. A package-wide yes/no would
hide differences between lookup, model, and derived paths.

| Package   | Historical output baseline                                                                    | 1.2 output behavior | 1.2 artifact integrity | 1.2 empirical support |
| --------- | --------------------------------------------------------------------------------------------- | ------------------- | ---------------------- | --------------------- |
| ethnicolr | Contract 1.0 recorded                                                                         | unassessed          | unassessed             | unassessed            |
| naampy    | Parallel column names and incomplete common metadata recorded                                 | unassessed          | unassessed             | unassessed            |
| pranaam   | Contract 1.1 score form recorded in [PR 60](https://github.com/appeler/pranaam/pull/60)       | unassessed          | unassessed             | unassessed            |
| outkast   | Package-specific status vocabulary recorded                                                   | unassessed          | unassessed             | unassessed            |
| instate   | Contract 1.1 composition form recorded in [PR 53](https://github.com/appeler/instate/pull/53) | unassessed          | unassessed             | unassessed            |

The
[previous adoption snapshot](https://github.com/appeler/appellation/blob/a2f383fa5978bb8ee201ba8661fd3155d32eaf81/ADOPTION.md)
contains the full historical matrix. Its calibration and evaluation entries were
broad claims rather than linked operation-specific assessments. They do not
establish support under the revised requirements. A missing assessment here does
not establish that a package's evidence is inadequate.

## Migration work

Every package must review the revised meanings in contract 1.2 before changing
`inference_contract_version`. This includes lookup versus estimate framing,
missing values, uncertainty metadata, and local-artifact identity. A version
bump alone is not migration. The items below are review targets; verify their
current implementation before making changes.

### outkast

Use the deterministic lookup as the first complete adoption. Define the filtered
SECC source, state/birth-year/surname unit, retained categories, and
denominator. Reconcile counts and proportions; document retention, suppression,
and reconstruction checks. Descriptive enumeration shares need no invented
sampling intervals. Any inference beyond those source cells needs separate
evidence.

The historical migration items were to replace `secc_*` status fields with
boolean `scored` and `abstained` plus `abstention_reason`; change
`insufficient_support` to `insufficient-evidence` and other reason tokens to
hyphens; adopt composition form; rename the `frame` argument to `data`; and
remove dead `outkast/utils.py` and its autodoc reference. Verify input collision
behavior against the contract as well.

The August record described an unpublished 2.0.0. Check current release state
before selecting a migration version; that old window is not an instruction to
publish now.

### naampy

Use a learned score as the second adoption. Define the female source-label share
among retained binary-label electoral records, exclusions, and record versus
name weighting. Audit normalized-name partitions and the calibration assessment
for the exact shipped artifact. Assess the exact lookup as a separate operation.

The historical migration items were to rename `score_target` to `target` and
`calibration_population` to `calibration_reference`, add missing common columns,
and add the DataFrame-and-column call form. Explain whether state and birth-year
conditioning is supported and how pooling changes the target.

Keep bootstrap intervals for model-card performance metrics with those metrics.
The old instruction to expose them as per-name `_lower` and `_upper` columns was
incorrect. Per-name intervals need a distinct target, procedure, and validation;
their absence alone does not violate 1.2.

### ethnicolr

Assess each Census lookup, voter-file model, and Wikipedia/Wikidata model
separately. Define their different label sources, populations, and weighting.
Review whether Census lookup intervals assume a broader population model; do not
describe enumeration counts as a probability sample without justification. Audit
evaluation splits against the intended unseen-name or source-transfer claim.

The historical migration items were to add `result_form`, replace the
package-owned inference-contract document with a versioned pointer here, and add
per-file SHA-256 verification rather than relying on a revision pin alone. Adopt
the current contract after reviewing all 1.2 requirements.

### pranaam

The August record describes a released 0.9.0 migration with score form,
canonical signature, explicit blank-name abstention, Monte Carlo dropout
summaries, and prior adjustment. Reassess calibration and label provenance for
the shipped model. Document dropout variability as such, and assess any stronger
interval claim separately. State prior-shift assumptions and preserve missing
scores through adjustment.

That release lacked independent second-model review. The record also notes a
subsequently fixed bug where prior adjustment converted an abstaining row's
missing score to 1.0. Verify the regression test and the artifact/release
containing the fix during reassessment. Offering dropout summaries alone does
not establish adequate uncertainty evidence.

### instate

The August record describes a released 3.0.0 migration with composition form, a
temperature-scaled state model, and census-based language mixtures. Assess the
lookup, state model, and language calculation separately. Document retained
surname-state cells, denominators, dates, and coverage.

For the language calculation, state the assumption behind substituting statewide
language shares for surname-specific shares within states. Separate reproducing
that mixture from validation against independently observed language data. The
old comparison of transformed versus direct model loss does not by itself
establish the substantive interpretation.

Review state-model calibration and uncertainty in held-out metrics. A per-name
interval mechanism is no longer a universal requirement; add one only with a
defined target and evidence for its interpretation.

## Applying the standard

Complete the outkast lookup record, then the naampy model record, before
extracting shared executable checks. Continue the remaining migrations against
the same requirements, with package-specific evidence. Existing instate and
pranaam contract implementations are examples of 1.1 behavior, not authoritative
implementations of 1.2.

Each package owns its implementation. Shared fixtures or a development helper
may follow demonstrated repeated checks; this revision introduces no runtime
dependency. The
[design-pattern comparison](<reports/Standards library design patterns.md>)
explains this choice and the conditions for revisiting it.
