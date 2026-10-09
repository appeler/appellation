# Package standards around claims and evidence

**Appellation should publish a versioned specification bundle, with optional
validation tools added when adoption demonstrates their value.** Established
projects separate the agreement from its implementations: normative requirements
define meaning, schemas describe machine-readable structure, tests exercise
behavior, and evidence supports substantive claims. Appellation needs all these
responsibilities, but does not yet need separate software for each. Its
immediate product is authoritative prose, an operation-level evidence template,
concrete conformance examples, and an adoption register. A skill guides
maintainers through applying that product to a specified version. Passing output
checks and establishing empirical support remain separate findings.

## The local fleet already demonstrates these patterns

The local py-canon, r-canon, and preen checkouts were inspected on 2026-10-09,
including implementation and test files. These are design observations, not
claims that their full test suites or downstream fleets passed an audit.

| Project  | What it actually packages                                                                                               | Lesson for Appellation                                                                                                               |
| -------- | ----------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| py-canon | Normative prose, reusable workflows, Copier templates, and Python code for shared Sphinx configuration and fleet triage | A standards project can contain software when consumers need executable behavior; being a standard alone does not require a library. |
| r-canon  | Normative prose, reusable workflows, adoption and drift scripts, and shared lint configuration                          | Reuse ecosystem tools and add only the missing coordination. A standards repository need not be an installable package.              |
| preen    | A separate conformance/adoption CLI and a skill directing agents to that CLI                                            | Keep requirements, executable checks, and agent instructions distinct. Reuse the existing implementation of package checks.          |

Sources are the inspected revisions of
[py-canon](https://github.com/gojiplus/py-canon/blob/a227c2058f0c3559b05edb0118e5cfff28de718b/README.md),
[r-canon](https://github.com/gojiplus/r-canon/blob/604ded50f02e7dc09387fe00fa5095732ec78b89/README.md),
and
[preen's skill](https://github.com/gojiplus/preen/blob/b88c258107d409a60c0dd6294eeb7317d7a76cf5/skills/preen/SKILL.md).

The implementation details matter. r-canon parses workflow job references and
tests that a reference appearing only in a comment does not pass. Preen's runner
rejects unknown check names rather than reporting success after selecting
nothing. Its issue records separate the finding from an optional proposed fix.
Appellation should likewise test actual returned behavior, report executed and
skipped checks, and keep diagnosis separate from changes to scientific choices.
([r-canon regression test](https://github.com/gojiplus/r-canon/blob/604ded50f02e7dc09387fe00fa5095732ec78b89/tests/testthat/test-workflow-references.R),
[preen runner](https://github.com/gojiplus/preen/blob/b88c258107d409a60c0dd6294eeb7317d7a76cf5/src/preen/checks/runner.py),
[preen issue model](https://github.com/gojiplus/preen/blob/b88c258107d409a60c0dd6294eeb7317d7a76cf5/src/preen/checks/base.py))

Reference shared logic where possible; when files must be copied, check for
drift. py-canon documents that lockfiles can retain old commits despite a moving
major tag, and that recording a moving tag as the Copier baseline has prevented
intended updates. Preen also tests its copied adoption defaults against the
canonical template. For Appellation, an assessment should record resolved
revisions, and future shared cases should test every implementation against the
same versioned expectations. A shared helper remains an option when it removes
demonstrated duplication; a skill should call it rather than recreate its logic.
([py-canon propagation notes](https://github.com/gojiplus/py-canon/blob/a227c2058f0c3559b05edb0118e5cfff28de718b/README.md),
[preen synchronization tests](https://github.com/gojiplus/preen/blob/b88c258107d409a60c0dd6294eeb7317d7a76cf5/tests/test_canon_template_sync.py))

This comparison also exposed a conflict in Appellation's artifact rule: allowing
any bundle below 5 MB in a wheel would admit learned weights that py-canon
forbids at every size. R01 now limits that allowance to lookup tables and
non-weight metadata. Preen already checks Python runtime assets, so a future
Appellation tool should reuse that check and add result semantics and
evidence-reference validation. The present pandas contract remains Python
specific; learning from r-canon does not make R packages conformant.
([py-canon runtime assets](https://github.com/gojiplus/py-canon/blob/a227c2058f0c3559b05edb0118e5cfff28de718b/STANDARD.md#runtime-assets),
[preen runtime-assets check](https://github.com/gojiplus/preen/blob/b88c258107d409a60c0dd6294eeb7317d7a76cf5/src/preen/checks/runtime_assets.py))

## Established projects separate authority from implementation

OpenAPI explicitly makes specification text authoritative over its informational
JSON Schemas and warns that schemas cannot detect every violation. It also
separates the specification version from the API description's own version.
These are useful precedents for Appellation: **a validator implements selected
rules; it does not replace the standard**, and package releases must not stand
in for standard versions.
([OpenAPI specification](https://spec.openapis.org/oas/v3.2.1.html),
[publication index](https://spec.openapis.org/oas/))

| Project                     | Packaging pattern                                                   | Application to Appellation                                                               |
| --------------------------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| OpenAPI                     | Authoritative prose with separately published schemas               | Keep interpretation in normative documents; identify which rules tools check.            |
| JSON Schema Test Suite      | Language-independent cases with expected outcomes; external runners | Publish valid and invalid examples before extracting shared Python helpers.              |
| Data Package / Frictionless | Portable data standard and a separate implementation framework      | Distinguish descriptor validity, actual data checks, and package behavior.               |
| BIDS                        | Specification, shared schema, examples, and validator               | Coordinate changes across requirements and examples; defer custom schema infrastructure. |
| Model Cards                 | Structured disclosure of uses, data, evaluation, and limitations    | Require operation-specific evidence without treating disclosure as proof.                |

The JSON Schema suite leaves runners to implementations, while Frictionless
distinguishes checking a descriptor from checking the resource it describes.
BIDS goes further: shared declarative definitions support documentation and
validation, reducing duplicated maintenance. That architecture solves a
demonstrated coordination problem at BIDS's scale; it does not establish that
Appellation needs a custom schema language.
([JSON Schema Test Suite](https://github.com/json-schema-org/JSON-Schema-Test-Suite),
[Frictionless validation](https://framework.frictionlessdata.io/docs/guides/validating-data.html),
[BIDS schema](https://bids-specification.readthedocs.io/en/stable/appendices/schema.html))

## Scientific requirements need evidence beyond valid fields

A schema can require a denominator description, calibration report, and artifact
digest. It cannot establish that the denominator matches a research question,
that the evaluation avoids leakage, or that a named calibration procedure
worked. Model Cards supplies a useful reporting pattern: describe intended uses,
training and evaluation data, relevant population factors, quantitative
performance, and limitations. The proposal also recognizes that reporting
quality depends on its creators. **Structured disclosure improves
inspectability; empirical claims still need examination.**
([Model Cards](https://arxiv.org/html/1810.03993v2))

Appellation therefore uses common descriptive requirements with obligations
conditional on the operation. Every lookup, learned estimate, or derived
composition identifies the target quantity, observation unit, denominator, label
provenance, geography, period, exclusions, weighting, and supported inputs.
Lookup evidence addresses source counts, suppression, coverage, and the
interpretation of reported proportions. Learned estimates require evaluation
splits matched to their generalization claims, held-out calibration diagnostics
where probabilities are claimed, and performance alongside abstention coverage.
Derived compositions disclose their transformation and assumptions and evaluate
the resulting quantity when making empirical accuracy claims.

Uncertainty also follows the quantity. An interval for model performance cannot
become an interval for one name's score. Prediction-set coverage and uncertainty
in an aggregate share are different claims. Enumeration counts do not alone
justify a sampling interval. These are Appellation's substantive requirements,
informed by the agreed corrections; the engineering precedents do not supply
universal thresholds or methods for demographic inference.

## The repository packages a reviewable agreement

The chosen application keeps [CONTRACT.md](../CONTRACT.md) and
[STANDARD.md](../STANDARD.md) normative. The [evidence template](../EVIDENCE.md)
prompts package maintainers to supply operation-specific documentation, while
[conformance examples](../CONFORMANCE.md) show observable requirements and
failure cases. [ADOPTION.md](../ADOPTION.md) records assessed operations and
links to package-owned implementation, tests, artifacts, and reports. README
guidance explains how these pieces fit together. Examples and templates guide
implementation; they do not silently introduce obligations beyond the normative
documents.

Conformance reporting separates output behavior, provenance, and empirical
support. Behavioral cases include preserved rows, explicit abstention, missing
estimates on unscored rows, bounded finite values, and valid composition totals.
Some require input/output comparisons or pandas-specific assertions; a JSON
example alone cannot establish those properties. Synthetic examples demonstrate
mechanics and must not be presented as demographic ground truth. Likewise, this
repository revision does not demonstrate that downstream packages have migrated.

Each adoption record should identify the standard version, package release or
commit, operation, artifact digest, evidence revision, and review date. BIDS
similarly distinguishes its standard version from generating software and source
datasets. Appellation should distribute immutable tagged releases and explain
changed obligations in release notes; changed empirical requirements must not
disappear under an unchanged conformance claim.
([BIDS dataset description](https://bids-specification.readthedocs.io/en/stable/modality-agnostic-files/dataset-description.html))

## Implementation tools follow demonstrated repetition

The next design test is complete adoption for a lookup and a learned model.
Their evidence will show which fields remain stable and which checks actually
repeat. After that, a small metadata schema can use an existing JSON Schema
dialect; a Python checker can become an optional development dependency if
multiple packages need it. Its version and supported standard versions should
remain distinct. Neither step requires a runtime dependency, a custom validation
engine, or copying the complete standard into a skill.

## Applying the pattern across the four repositories

The consistency pass uses each project's existing structure rather than
requiring every standards repository to become a package or acquire a CLI. The
common obligations are explicit authority, versioned requirements,
implementation traceability, honest assessment scope, behavior tests, reviewed
adoption, and documented change consequences.

| Repository  | Application                                                                                                                                                                          |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Appellation | Normative contract and standard; separate evidence and examples; operation-scoped adoption; explicit limits on mechanical and scientific conformance                                 |
| py-canon    | Authority and verification map in STANDARD.md; propagation and changelog rules aligned with tooling; successful candidate CI required for major-tag promotion                        |
| r-canon     | Authority and audit-scope rules in STANDARD.md; consumer-run limitations explicit; successful candidate CI required for major-tag promotion; current major used for manual promotion |
| preen       | Policy authority remains py-canon; CLI reports exclusions and executed scope and rejects empty assessments; contribution and skill guidance point to the same policy                 |

The promotion and empty-run changes have behavioral regression tests. Their
verification does not substitute for executing GitHub-hosted promotion or
running every downstream consumer. The tools retain their own versions;
assessments identify the exact standard and implementation revisions used.
