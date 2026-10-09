# Appellation

Appellation defines result behavior and evidence requirements for the appeler
name-analysis packages. It specifies what an output means, when an operation
must abstain, and what evidence supports its statistical claims. Output
conformance and empirical support are separate assessments.

This is a standards repository. Packages implement the contract and keep their
own tests and evidence; users do not install Appellation as a runtime
dependency. A skill may guide an audit against a specified version, but neither
following a skill nor passing a schema check establishes scientific validity.

## Contents

| Document                                                          | Purpose                                                                                                            |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [CONTRACT.md](CONTRACT.md)                                        | Normative result contract, version 1.2: columns, missing values, invariants, abstention, and provenance            |
| [STANDARD.md](STANDARD.md)                                        | Normative version 1.2 requirements for data definitions, APIs, calibration, uncertainty, evaluation, and artifacts |
| [EVIDENCE.md](EVIDENCE.md)                                        | Template for an operation's package-owned, versioned evidence record                                               |
| [CONFORMANCE.md](CONFORMANCE.md)                                  | Boundary examples for implementation tests and the limits of mechanical checks                                     |
| [ADOPTION.md](ADOPTION.md)                                        | Historical migration facts, current assessment status, and remaining work                                          |
| [Design patterns](<reports/Standards library design patterns.md>) | What OpenAPI, JSON Schema, Frictionless, and BIDS suggest for organizing this repository                           |

## Packages and data in scope

Ethnicolr, naampy, pranaam, outkast, and instate work with different reference
data: Census name tables, voter records, biographical sources, labeled name
corpora, and filtered administrative aggregates. Their targets and observation
units differ. The standard requires each operation to define its source, labels,
denominator, geography, period, and intended population. It does not make these
sources interchangeable.

A lookup, learned model, and derived composition have different evidence
obligations. Result form describes the output shape; it does not determine which
statistical assumptions apply. Aggregate research still needs validation for its
target population.

[py-canon](https://github.com/gojiplus/py-canon) governs how packages are built,
tested, and released. Appellation governs result meaning and the evidence behind
it. Packages outside the name-analysis family, such as indicate, follow py-canon
without adopting this standard.

[preen](https://github.com/gojiplus/preen) supplies Python conformance and
adoption tooling; Appellation should reuse its package checks.
[r-canon](https://github.com/gojiplus/r-canon) provides a parallel design
precedent: standards plus shared workflows, using existing R tools for checks.
It does not make the current pandas contract an R API standard.

## Adoption and changes

Start with an operation's data and target definition, implement the applicable
requirements, exercise actual behavior locally, and record the evidence. Link
the assessment from ADOPTION.md with separate output, artifact, and
empirical-support statuses. No package has been reassessed against 1.2 in this
repository.

The contract and standard use a shared specification version and change through
pull requests. Their version is distinct from package releases, artifact
revisions, and evidence revisions. Cite an immutable Appellation commit or
release when assessing conformance. Changes include rationale, affected
operations, examples, and migration consequences; the contract's history records
semantic changes. There are no compatibility shims. Packages emit
`inference_contract_version` only after adopting that version's output rules;
the field is not a certificate of empirical support.

The contract originated in ethnicolr as version 1.0 and moved here at 1.1.
Version 1.2 replaces the generic uncertainty bar with requirements matched to
each quantity and separates historical adoption claims from reviewed evidence.
Shared schemas or executable fixtures should follow stable, repeated
requirements demonstrated in real migrations.

## Maintaining the specification

CONTRACT.md and STANDARD.md are authoritative. Templates, examples, research
notes, skills, and future validators are supporting material. Resolve a
contradiction against the explicit requirement and its version; a checker does
not silently redefine the rule. Update affected examples and adoption guidance
when requirements change.

Changes should include a conforming example, a violating example, and the
migration consequence. Documentation-only revisions run Markdown lint,
formatting, and link checks. Executable checks, when introduced, need behavioral
regressions and a real consumer test. A text search for a required field is not
proof that a package returns it correctly.

Assessments record immutable standard, implementation, artifact, and evidence
revisions, plus executed and unavailable checks. Membership in ADOPTION.md is
not conformance. Preserve historical assessments under their original versions;
changed obligations require reassessment. Restrict automatic fixes to mechanical
changes whose meaning is established; scientific choices require evidence.
