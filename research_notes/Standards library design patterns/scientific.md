# Scientific standards and evidence patterns

## What belongs in the standard, schema, validator, and evidence?

### Takeaway

BIDS demonstrates how a scientific standard can distribute documents, metadata
definitions, examples, and validation tools without making every consumer depend
on a shared runtime library. Appellation should adopt the separation of
responsibilities, scaled to five packages.

### Cited Findings

- The BIDS schema is a declarative representation of the standard. It supplies
  shared definitions for rendering documentation and validating datasets,
  reducing inconsistency between independently maintained descriptions and
  checks. It is distributed as YAML source and a compiled JSON document. —
  [BIDS schema](https://bids-specification.readthedocs.io/en/stable/appendices/schema.html)
- The schema-driven validator checks filenames, JSON metadata, TSV columns and
  values, and additional encoded rules. BIDS reports that previously updating a
  separate JavaScript implementation became a bottleneck for extensions; shared
  definitions reduced duplicated work. —
  [BIDS Validator 2.0](https://bids.neuroimaging.io/blog/2024/11/13/bids-validator-2.html)
- Validation distinguishes errors that prevent compliance from warnings about
  noncritical issues, such as missing recommended fields. —
  [BIDS validation](https://bids.neuroimaging.io/tools/validator.html)
- Model Cards proposes reporting intended uses, evaluation procedures, dataset
  provenance and preprocessing, relevant population factors, quantitative
  results, and caveats. It explicitly says a card's usefulness and accuracy
  depend on its creators' integrity. —
  [Model Cards for Model Reporting](https://arxiv.org/html/1810.03993v2)

### Inferences

- Use normative prose for statistical obligations and interpretation; structured
  metadata for identifiers and required descriptive fields; tests for observable
  behavior; and empirical reports for scientific evidence. A field can be
  present and type-correct while its substantive claim is unsupported.
- Declare conformance separately for output behavior, artifact/provenance
  requirements, and reviewed statistical evidence. Do not label a result
  scientifically valid merely because its shape passes validation.
- A JSON evidence schema may be useful after a human-readable template has been
  exercised. Avoid copying BIDS's custom schema language or building a generator
  before duplication becomes a demonstrated maintenance problem.

### Gaps

- The reviewed BIDS pages describe structural compliance, not a general
  certification framework for scientific validity. That boundary for Appellation
  is a design recommendation, not a claim that BIDS provides such certification.

## How should versions, extensions, and conformance evidence work?

### Takeaway

Pin the standard independently from software and data artifacts. Require
concrete examples and evidence when extending it.

### Cited Findings

- BIDS requires a dataset's `BIDSVersion`; derived datasets additionally require
  `GeneratedBy` provenance. Software version and source-dataset references have
  separate fields. —
  [Dataset description](https://bids-specification.readthedocs.io/en/stable/modality-agnostic-files/dataset-description.html)
- Compiled schemas identify both the schema and specification versions, and
  version-specific schema snapshots are available. —
  [BIDS schema README](https://github.com/bids-standard/bids-specification/blob/master/src/schema/README.md)
- BIDS extension proposals progress from draft through proposed to merged, with
  review between phases. Required deliverables include specification text,
  schema changes, and example datasets; the process recommends creating examples
  early to expose complexity and feasibility problems. —
  [BEP process](https://bids.neuroimaging.io/extensions/process.html)

### Inferences

- Record the Appellation version, package version/commit, artifact digest,
  operation, evidence revision, test command/result, and review date in adoption
  records. A package-wide “yes” loses essential scope.
- Keep proposed requirements visibly draft until accepted. For five packages, a
  reviewed pull request containing the rationale, affected operations, examples,
  and migration consequences is sufficient; a large committee workflow is
  unnecessary.

### Gaps

- This research did not verify compatibility behavior across all historical BIDS
  versions. Appellation needs its own explicit rule for when changed obligations
  require a new major standard version.

## What minimum scientific evidence should Appellation require?

### Takeaway

Use one compact operation-level evidence template, with obligations conditional
on whether the operation is a lookup, learned estimate, or derived composition.

### Cited Findings

- Model Cards distinguishes training data from evaluation data and asks for
  intended uses, challenging evaluation scenarios, performance across relevant
  factors, and uncertainty for reported performance metrics. —
  [Model Cards](https://arxiv.org/html/1810.03993v2)

### Inferences

- Every operation needs its target quantity, observation unit, denominator,
  label provenance, geography/time, exclusions, weights, input support, and
  source limitations.
- Learned estimates need leakage controls matched to the intended generalization
  claim, held-out calibration diagnostics, performance and abstention coverage,
  and explicit limits on population transfer.
- Uncertainty must identify its quantity and assumptions: variation in model
  performance, uncertainty around a name-specific score, prediction-set
  coverage, and uncertainty in an aggregate share are different objects.
  Enumeration alone does not supply a sampling model.
- Derived compositions need the transformation and its assumptions stated
  explicitly, including any conditional independence or within-group homogeneity
  approximation. Evaluation should test the resulting composition when that is
  the intended use.
- Evidence may document an unresolved limitation honestly; this is distinct from
  satisfying a requirement for a particular scientific claim.

### Gaps

- These scientific requirements adapt the user's agreed corrections; the
  comparative sources do not supply universal calibration thresholds, split
  rules, or uncertainty methods for name-analysis data. Those must be justified
  for each operation.
