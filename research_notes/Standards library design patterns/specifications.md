# Specification-first design patterns

## What is normative, and what is executable?

### Takeaway

Publish a specification that defines meaning, then supply executable checks for
the subset of requirements that software can test. A validator is an
implementation aid, not the final authority on statistical validity.

### Cited Findings

- OpenAPI explicitly makes its specification text authoritative when an
  informational JSON Schema disagrees. It identifies an authoritative published
  rendering and uses uppercase requirement terms under BCP 14.
  [OpenAPI 3.2.1, status and sections 1 and 4](https://spec.openapis.org/oas/v3.2.1.html)
- OpenAPI publishes multiple dated schema revisions separately from
  specification releases and warns that schemas cannot detect every
  specification violation.
  [OpenAPI publication index](https://spec.openapis.org/oas/)
- The JSON Schema Test Suite describes specified behavior using JSON cases,
  while implementers supply their own language/framework-specific runners. It
  explicitly distinguishes conformance cases from examples of recommended
  writing style.
  [Official test-suite README](https://github.com/json-schema-org/JSON-Schema-Test-Suite)

### Inferences

- Keep `CONTRACT.md` and `STANDARD.md` authoritative. Mark examples, evidence
  templates, and implementation guidance as informative; give requirements
  stable IDs so tests and review evidence can refer to them.
- Make “metadata validates,” “behavior tests pass,” and “empirical evidence
  supports this use” separate findings. Requiring a calibration report is
  mechanically checkable; judging its design and adequacy requires review.
- For Appellation, release an inspectable bundle of prose, templates, and
  fixtures. Neither a wheel nor a runtime import is necessary to distribute that
  bundle.

### Gaps

- These engineering specifications do not establish requirements for demographic
  inference. Their architecture transfers; their subject-matter rules do not.

## How are versions, dialects, and implementations separated?

### Takeaway

Identify the rules an artifact follows separately from the artifact itself. Use
an existing schema dialect rather than inventing one.

### Cited Findings

- OpenAPI's `openapi` field identifies the specification version independently
  of `info.version`, which identifies the API-description version. Patch
  versions clarify or correct specification text rather than adding features.
  [OpenAPI sections 2.1 and 4.1.1](https://spec.openapis.org/oas/v3.2.1.html)
- JSON Schema's `$schema` declares the dialect; omitting it can leave
  interpretation to implementation assumptions. Custom validation keywords
  require implementations to understand their semantics, reducing
  interoperability.
  [JSON Schema dialect guidance](https://json-schema.org/understanding-json-schema/reference/schema)
- The official JSON Schema suite has directories for specification versions and
  cases containing a description, schema, input data, and expected validity. The
  runner is external.
  [Official test-suite structure](https://github.com/json-schema-org/JSON-Schema-Test-Suite)

### Inferences

- Record the Appellation version, package release, model/table hash, and
  evidence revision separately. A package version alone does not identify the
  standard or evaluated artifact.
- If metadata schemas are added, declare standard Draft 2020-12. Put
  Appellation's standard version in an ordinary metadata property; it is not the
  JSON Schema dialect.
- Publish immutable tagged specification releases and a change log. A clarified
  requirement and a newly imposed empirical obligation should not silently share
  an unchanged version.
- Define operation profiles such as lookup, learned estimate, and derived
  composition. Do not confuse these applicability rules with schema dialects or
  separate standards.

### Gaps

- No source establishes the appropriate Appellation version number or guarantees
  that existing packages meet a revised standard; those require repository
  decisions and audits.

## What minimal structure suits Appellation?

### Takeaway

Start with a small specification bundle and adoption evidence. Extract shared
validation code only after actual implementations demonstrate recurring needs.

### Cited Findings

- JSON Schema demonstrates that shared fixtures can be distributed independently
  of implementation libraries. Its repository also checks the test suite itself,
  separating fixture correctness from implementation correctness.
  [Official test suite](https://github.com/json-schema-org/JSON-Schema-Test-Suite)

### Inferences

- Suggested artifacts: existing `CONTRACT.md` and `STANDARD.md`;
  `templates/evidence.md`; `examples/lookup-evidence.md`;
  `conformance/README.md`; later, `conformance/cases/abstention.json` and
  `schemas/evidence.schema.json` if structured records prove useful.
- Each requirement should state scope, test or review method, and required
  evidence. Examples: preserved rows → package test; source denominator → data
  documentation; calibration → held-out diagnostics; transfer → external
  evaluation or an explicit untested limitation.
- `ADOPTION.md` should link package-owned reports and tests with assessed
  versions, rather than certify an entire package with a yes/no cell.
- Tradeoff: prose plus templates keeps maintenance low but depends on review.
  Schemas catch omissions but add synchronization work and can encourage
  superficial compliance. Shared test helpers become worthwhile when several
  packages demonstrably repeat stable behavior checks.

### Gaps

- A complete adoption audit and concrete repeated test implementations are
  needed before choosing a shared Python conformance package.
