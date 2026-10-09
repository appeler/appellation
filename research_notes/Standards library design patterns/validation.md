# Schema and validator packaging patterns

Research checked October 9, 2026. Recommendations below are design inferences
for Appellation, not claims of existing compliance.

## What can a schema validate?

### Takeaway

Separate descriptor validity, actual data validity, executable behavioral
checks, and empirical evidence. They answer different questions and should
produce separate conformance claims.

### Cited Findings

- Frictionless explicitly distinguishes validating schema metadata
  (`Schema.validate_descriptor`) from validating a resource's data and metadata.
  Its framework also accepts custom checks for requirements beyond the built-in
  checks. A valid description is therefore not evidence that the described file
  passes validation. —
  [Frictionless validation guide](https://framework.frictionlessdata.io/docs/guides/validating-data.html)
- Table Schema declares column types, missing-value interpretation, numeric
  bounds, categorical choices, primary keys, and foreign keys. It is independent
  of programming language and implementation. The current specification
  recommends the explicit 2.0 profile; its default profile remains 1.0. —
  [Table Schema](https://datapackage.org/standard/table-schema/)
- Pandera supports column, grouped, and whole-dataframe checks. Its custom
  checks drop null values by default unless configured otherwise, which matters
  when testing abstention behavior. —
  [Pandera checks](https://pandera.readthedocs.io/en/stable/checks.html)

### Inferences

- Use JSON Schema for an artifact/evidence descriptor: required reference
  population, denominator, observation unit, operation, source revision,
  evaluation links, and uncertainty interpretation. Passing means required
  claims are present and structurally well formed, not that those claims are
  true.
- Test actual outputs separately: finite bounded scores, category membership,
  explicit abstention, and normalized compositions. Cross-column constraints
  such as abstention implying missing scores require explicit rules; input-row
  preservation requires comparing input and output. Hash verification requires
  reading the referenced bytes.
- Calibration cannot be established from output shape or a metadata field
  declaring `calibrated=true`. Evidence must connect predictions to held-out
  outcomes under a specified population, split, weighting, and diagnostic
  procedure. A reproducible evaluation can check numerical claims; a reviewer
  must still assess assumptions and applicability.

### Gaps

- No runtime compatibility between a particular Frictionless release and all
  Table Schema 2.0 features was tested. If adopted, pin and test the exact
  supported profile instead of assuming specification and implementation
  versions coincide.

## How should fixtures and implementations relate to the standard?

### Takeaway

Publish small, versioned examples with expected outcomes independently of a
particular validator. Implementations should demonstrate that they interpret the
rules consistently.

### Cited Findings

- The JSON Schema Test Suite is language agnostic. Each case pairs a schema with
  described instances and expected validity booleans. Cases are organized by
  specification version. Its documented limitations explicitly recognize that a
  test suite can only test behaviors expressible by its test mechanism. —
  [JSON Schema Test Suite](https://github.com/json-schema-org/JSON-Schema-Test-Suite)
- Frictionless's validation reports identify errors and their locations,
  supporting inspectable failures instead of a single unexplained pass/fail. —
  [Validation reports](https://framework.frictionlessdata.io/docs/guides/validating-data.html)

### Inferences

- Add fixtures for each Appellation output form: a valid result, valid
  abstention, and invalid variants for missing provenance, unsupported reason
  code, nonfinite score, inconsistent label, and composition totals outside a
  declared tolerance. Include repeated input rows and empty inputs in behavioral
  tests.
- Record which rule each fixture exercises and its expected failure. Fixtures
  should use synthetic names/category labels; they are demonstrations of
  mechanics, not demographic ground truth.
- Treat pandas dtypes/index behavior as a Python binding concern. A JSON fixture
  can represent a logical boolean but cannot prove that an implementation
  returns the required pandas dtype or preserves an index.

### Gaps

- Fixture coverage cannot establish scientific adequacy, complete software
  correctness, or generalization beyond the tested examples.

## What should Appellation package now and later?

### Takeaway

Package a versioned specification bundle now; introduce a Python conformance
helper only after more than one real adopter demonstrates shared executable
needs.

### Cited Findings

- Data Package publishes an open standard for describing collections of data
  files, while Frictionless publishes a Python framework implementing
  data-management operations. This illustrates separating a portable agreement
  from one implementation. — [Data Package](https://datapackage.org/);
  [Frictionless framework](https://framework.frictionlessdata.io/)

### Inferences

- Now: maintain normative prose, an evidence template, stable requirement
  identifiers, examples/fixtures, an adoption register linking to actual
  evidence, and a changelog. Publish tagged repository releases so packages can
  cite an immutable standard version. Keep illustrative examples clearly
  distinct from audited package evidence.
- Add a small descriptor schema only when its required fields have settled, with
  positive and negative tests using a standard JSON Schema validator. Ordinary
  package tests can cover runtime behavior; no universal validator framework is
  necessary yet.
- Later: extract repeated output checks into an optional development dependency.
  Version that implementation independently, state which standard versions it
  checks, and report unsupported checks honestly. Keep empirical review outside
  the validator's certification claim.
- Do not require an Appellation runtime dependency, copy the complete standard
  into a skill, or mandate Frictionless/Pandera solely because they supplied
  useful patterns. The skill should orchestrate adoption against a specified
  release.

### Gaps

- The five downstream packages were not audited in this research; selecting a
  common Python dependency requires inspecting their actual result shapes,
  dependencies, and test workflows.
