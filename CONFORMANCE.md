# Conformance examples

These examples illustrate Appellation 1.2 requirements in
[CONTRACT.md](CONTRACT.md) and [STANDARD.md](STANDARD.md). The normative
documents take precedence. This is a test-design guide, not an executable
validator or evidence that a package passes.

Packages should exercise applicable cases against their actual public functions
and artifacts. Use synthetic names and categories for contract tests; evaluate
statistical claims on the declared reference data. Unspecified metadata in these
abbreviated examples is assumed valid.

## Result behavior

| Case                                                                                | Expected behavior                                                                                   | Requirement   |
| ----------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- | ------------- |
| Label distribution A=0.7, B=0.3; no abstention                                      | Label A and predicted probability 0.7                                                               | C04, C07      |
| Same distribution; label B or predicted probability 0.8                             | Invalid result                                                                                      | C07           |
| Label distribution A=0.51, B=0.49; declared decision rule abstains                  | Keep the distribution, mark scored and abstained, set reason, leave both label fields missing       | C02, C08      |
| Score 0.5 on a supported input                                                      | Valid score; no label columns, no forced classification                                             | C09, P03      |
| Score result also includes a null predicted-label column                            | Invalid schema; omit label-only columns                                                             | C09           |
| Descriptive composition with counts A=30, B=70 and denominator 100                  | Proportions 0.3 and 0.7; no label-only columns                                                      | C04, C10      |
| Same counts and denominator with proportions 0.4 and 0.6                            | Invalid descriptive composition even though proportions sum to one                                  | C10           |
| Scored row with a missing component, infinity, or total 1.2                         | Invalid result                                                                                      | C04           |
| Missing name                                                                        | Keep row, abstain with missing-name, set scored and script_supported false, leave estimates missing | C01, C02      |
| Unsupported script with a populated score                                           | Invalid result                                                                                      | C01, C03      |
| Exact lookup miss                                                                   | Abstain; no silent model or prior fallback                                                          | A01, P01      |
| Previously unseen name within a model's evaluated domain                            | May be scored; novelty alone does not require abstention                                            | P01           |
| Suppressed lookup cell                                                              | Abstain with insufficient-evidence; do not expose suppressed counts                                 | C01, R03      |
| Successful row with a nonmissing abstention reason                                  | Invalid result                                                                                      | C02           |
| Prior adjustment applied to a missing score                                         | Score stays missing and abstention is preserved                                                     | C06           |
| Prior adjustment moves a supported label distribution across its decision threshold | Reapply the documented decision rule and update all related fields consistently                     | C06, C07, C08 |
| Score or composition marked both scored and abstained                               | Invalid state; only label form supports scored abstention                                           | C09, C10      |
| Interval with one missing endpoint or reversed bounds                               | Invalid uncertainty output                                                                          | C11           |
| Probability interval for a missing score                                            | Both endpoints missing                                                                              | C01, C11      |

## API and artifacts

| Case                                                                 | Expected behavior                                                                           | Requirement                  |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | ---------------------------- |
| Empty DataFrame with the required input columns                      | Empty result with full applicable schema and preserved index                                | A02                          |
| Repeated indices, mixed supported and unsupported rows               | Preserve all rows, order, and indices; input remains unchanged                              | A02                          |
| Input contains scored and input_scored columns                       | Preserve both inputs; move the reserved name to an unused suffixed input name               | A02, contract common columns |
| Missing or duplicate input columns                                   | Raise a programming-error exception                                                         | A03                          |
| Well-formed context outside the artifact's supported states or years | Preserve rows and abstain with unsupported-context                                          | A03                          |
| Local mirror with valid files                                        | Verify contents and report local origin plus immutable content identity                     | C05, R01                     |
| Corrupted artifact or manifest unanchored to a trusted identity      | Fail verification before returning estimates                                                | R01                          |
| Changing model revision from a commit to main                        | Invalid provenance                                                                          | C05                          |
| Learned weights below 5 MB included in the wheel                     | Invalid packaging; the size allowance applies only to lookup tables and non-weight metadata | R01                          |

## Limits of mechanical checks

These cases need empirical or methodological review, even when every output row
has the correct schema:

| Evidence presented                                                   | What it does and does not establish                                                                                | Requirement |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ----------- |
| Temperature scaling was fitted                                       | Identifies a procedure; held-out diagnostics are still required                                                    | P02         |
| Bootstrap interval for mean test log loss                            | Quantifies uncertainty in that metric, not each name's probability                                                 | U02         |
| Narrow dropout quantiles                                             | Describes stochastic model variability; does not establish interval coverage                                       | U02         |
| Exact enumeration counts                                             | Establishes descriptive shares after reconciliation; does not supply a sampling design or eliminate coverage error | U01         |
| Random row split with the same normalized names in both partitions   | Does not establish unseen-name performance                                                                         | E01         |
| Language mixture evaluated against targets generated by that mixture | Tests reproduction of the formula, not agreement with observed surname-language data                               | E05         |
| Good source-population calibration                                   | Does not establish calibration or aggregate validity in a new population                                           | E03, E04    |

When multiple migrations reveal stable shared checks, these cases can become
portable fixtures with package-specific adapters. Keep expected behavior
versioned with the contract. A shared runner should execute the public behavior
being claimed; searching documentation for column names is not a conformance
test.
