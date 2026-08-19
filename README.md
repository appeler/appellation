# Appellation

Appellation is the result standard for the appeler name-analysis packages:
what a returned estimate means, what a package must do when it cannot
estimate, and how the artifacts behind an estimate are named, hosted, and
evaluated. An appellation contrôlée certifies what may carry a name; this
repository certifies what an appeler result may claim.

It is the semantic counterpart to
[py-canon](https://github.com/gojiplus/py-canon). py-canon governs how fleet
packages are built, tested, and released. Appellation governs what their
results mean. A package can pass every py-canon check and still return
uncalibrated scores, silently default unknown names, or drift its column
names away from its siblings. This repository exists to close that gap.

## Contents

| Document | What it rules on |
| --- | --- |
| [CONTRACT.md](CONTRACT.md) | Inference result contract, version 1.1: the columns every result carries, the invariants they satisfy, and the shared abstention vocabulary. |
| [STANDARD.md](STANDARD.md) | Everything around the result: function verbs and signatures, probability scale, the uncertainty bar, artifact formats and hosting, evaluation discipline, and intended-use framing. |
| [ADOPTION.md](ADOPTION.md) | Where each package stands against the standard, and the migration order. |

## Packages in scope

ethnicolr, naampy, pranaam, outkast, and instate. The contract was first
published inside ethnicolr as version 1.0; this repository is its home from
version 1.1 onward, and ethnicolr conforms to it rather than owning it.
Packages outside the name-analysis family (for example indicate) follow
py-canon but not this standard.

## How changes happen

The contract and standard are versioned together and changed by pull request
to this repository. There are no compatibility shims: when a rule changes,
each package adopts it in one breaking release, and the
`inference_contract_version` column in every result row says which rules
that row was produced under.
