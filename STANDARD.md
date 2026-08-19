# The appeler standard

This document rules on everything around a result that [CONTRACT.md](CONTRACT.md)
does not: how public functions are named and called, what probabilities and
uncertainty a package must offer, how artifacts are stored and verified, and
what evaluation a shipped model must have passed. Rules use must, should,
and may in their usual normative senses. Where a rule reverses something a
fleet package currently does, [ADOPTION.md](ADOPTION.md) carries the
migration item; there are no compatibility aliases or shims.

## Verbs and signatures

Public functions use two verbs. `lookup_*` returns values from a published
name table. `estimate_*` combines evidence or runs a statistical model. No
public function uses `get_`, `predict_`, `pred_`, or a bare noun. The verb
is a claim about epistemic status: a lookup reports what a source says, an
estimate is the package's own inference, and a user must be able to tell
which they are holding from the call site alone.

The canonical signature takes a DataFrame and the name of the column to
read, with every option keyword-only:

```python
estimate_target(data: pd.DataFrame, surname_column: str, *, ...) -> pd.DataFrame
```

Column arguments are `surname_column`, `first_name_column`, and
`name_column`, matching the input scope. The frame argument is `data`,
never `frame` or `df`. The function returns a copy of the input with result
columns appended, preserving row count, order, and index, and never mutates
its input. A package whose natural input is a single name column may also
accept `str`, `list[str]`, or `pd.Series` and return a fresh frame in input
order, provided the result columns and semantics are identical to the
DataFrame form.

Invalid calls raise; ambiguous rows abstain. A missing column, a duplicate
column, or an option value outside its domain is a programming error and
raises immediately. A name the package cannot score is data, and is
reported through the contract's abstention columns, never through an
exception and never through a silently missing or NaN row.

## Probabilities and abstention

Every probability, score, and proportion is on the 0 to 1 scale.
Percentages do not appear in results.

A package must not assign a default or prior distribution to an input it
cannot support. Unsupported inputs abstain, with a reason from the
contract's shared vocabulary. Returning the marginal distribution for an
unknown name is the failure mode this fleet was rebuilt to eliminate: it
manufactures a confident-looking answer precisely where the package knows
least.

Model scores must be calibrated before release, and `calibration_status`
must say how. Raw softmax output or raw logits are not a result. A package
that cannot yet calibrate a model does not expose that model's
probabilities.

## The uncertainty bar

The bar differs by what stands behind the number, because the honest
uncertainty statement differs.

A model package must always emit `calibration_status`, and must offer at
least one interval or set-valued mechanism: a conformal prediction set at a
requested coverage, a bootstrap or ensemble interval, or Monte Carlo
dropout summaries, using the contract's column naming. It must expose its
abstention threshold as a keyword option rather than a constant, and, for
label and score forms, should offer a prior-shift option so users can
adapt estimates to a target population with a different base rate.

A lookup package over a sample must offer sampling intervals, such as
Wilson intervals on count-based proportions. A lookup package over an
enumeration has no sampling uncertainty to report and must not invent one;
its obligation is to document coverage and any disclosure suppression, and
to abstain with `insufficient-evidence` where suppression bites.

## Artifacts

Runtime tables are typed Parquet validated against an explicit Arrow
schema at load. Vocabularies, labels, calibration statistics, and manifests
are schema-versioned JSON. CSV does not appear at runtime; it may appear in
training and acquisition pipelines, which are out of scope here.

Model weights and any artifact bundle above 5 MB live in the
`gojiberries` Hugging Face organization, pinned to a full 40-character
commit SHA, with a model card. The package downloads through the Hugging
Face cache and verifies a per-file SHA-256 manifest after download; the
revision pin fixes what the artifact is, and the hash catches a corrupted
or tampered copy the pin alone cannot. A `<PACKAGE>_MODEL_DIR` environment
variable selects a local mirror, and a result produced from a mirror must
say so in its provenance columns rather than reporting the Hub revision.

Bundles of 5 MB or less may ship inside the wheel instead, under the same
discipline: a manifest, a pinned hash checked at load, and the same
provenance columns. The threshold is a wheel-size budget, not a quality
tier; small deterministic tables are not second-class artifacts.

## Evaluation

A shipped artifact must have been produced under the package's declared
evaluation contract, and its held-out metrics must be published in the
repository. If the artifact predates the evaluation contract, the package
retrains or recalibrates before its next release; publishing metrics the
shipped weights never earned is worse than publishing none.

The rules that make those metrics mean something: split source rows before
any balancing or augmentation; learn vocabulary only from training rows;
keep calibration fitting and conformal evaluation disjoint; and report the
evaluation unit and weighting with every metric, because record-weighted
and name-weighted accuracy answer different questions.

## Framing

Package documentation states, prominently and in its own words, that
outputs are name-pattern estimates from a stated reference population and
do not establish a person's identity, ancestry, citizenship, religion,
caste, race, or ethnicity, and that individual profiling and
consequential-decision uses are out of scope. Function and column names
follow the same discipline: name the pattern estimated, not the identity
inferred, as in `estimate_muslim_name_pattern` rather than
`predict_religion`.

## Relationship to py-canon

[py-canon](https://github.com/gojiplus/py-canon) owns build backend, lint,
type checking, test and coverage gates, docs, CI, and release mechanics.
This standard assumes all of it and adds nothing to it. A rule that belongs
to how a package is built goes to py-canon; a rule that belongs to what a
result means goes here.

## Conformance

Conformance is assessed against [ADOPTION.md](ADOPTION.md), which records
each package's status and open migration items. A shared conformance test
suite that packages run in CI is the intended next step once two or more
packages conform; until it exists, each package's own contract tests, in
the style of naampy's `test_runtime_contract.py`, are the check.
