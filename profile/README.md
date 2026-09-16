# Amasario

**Soroban contract provenance, dependency and impact infrastructure.**

Amasario answers one question about a Soroban contract, and it answers it about a chain
rather than about a repository:

> What does this contract depend on, where did its deployed artifact come from, what
> evidence supports those relationships, and what else could be affected by a change?

The answers are documents: JSON, Markdown, DOT, GraphML and JUnit, each of which states
what was observed, what was inferred, what was verified, what is unknown and what failed
— as separate things that never merge.

## The layers

Four repositories, and the boundaries between them are the design rather than an
accident of packaging.

| Repository | Layer | What it is |
| --- | --- | --- |
| [`amasario-provenance-spec`](https://github.com/Amasario-Soroban-Click/amasario-provenance-spec) | **normative** | What Amasario means by contract identity, artifact identity, provenance, dependency, evidence, confidence and impact: 24 strict JSON Schemas, 10 taxonomies, 24 models, 18 rules, 17 fixtures, 13 deterministic vectors. It performs no analysis. |
| [`amasario-provenance-engine`](https://github.com/Amasario-Soroban-Click/amasario-provenance-engine) | **execution** | The Rust CLI and libraries that consume the specification: contract inspection, network observation, evidence collection, provenance verification, dependency discovery, graph construction, impact analysis, snapshots and reports. Read-only, with no command that submits a transaction and no key handling anywhere. |
| [`amasario-explorer`](https://github.com/Amasario-Soroban-Click/amasario-explorer) | **presentation** | A browser over the engine's own documents — dependency graphs, provenance chains, snapshot diffs, verification scope and the reference contract's decoded interface. |
| [`amasario-docs`](https://github.com/Amasario-Soroban-Click/amasario-docs) | **documentation** | The cross-repository documentation: how the layers fit, what each refuses to claim, and how to contribute to any of them. |

A specification with one implementation can be whatever that implementation does. Keeping
the normative model separate from the engine keeps it small enough to depend on and
checkable without trusting any particular program — and it means a change that would alter
what a document *means* is detectable as a schema failure rather than as a quietly
different analysis result.

## What Amasario refuses to claim

The vocabulary is deliberately narrow, and the narrowness is the product.

**Amasario is not a security scanner.** No output is a security opinion. No term in any
taxonomy means "trustworthy", "safe" or "vulnerable", and none of those words may be
emitted or implied. `VERIFIED` means *the stated evidence is consistent with the stated
claim* — evidence completeness, never the trustworthiness of a contract.

**An address is not an identity.** A Soroban contract address survives an upgrade
unchanged and says nothing about origin, so no document here can represent "the contract
at `C...`" as a complete identity.

**A claim without evidence is not a claim.** A confidence level that names no evidence is
not constructible. A dependency that does not state how it was established does not
validate — declared, resolved, observed, embedded, configured or attested, never quietly
inferred and dressed up.

**A contradiction must be representable.** When a claimed source revision rebuilds to a
different digest from the deployed module, the only possible answers are `CONFLICTING` or a
bug. An incorrect `VERIFIED` is the most damaging output this system can produce, because
it is the one that stops a reader looking further.

**A bounded search must say it was bounded.** "The search stopped" and "there was nothing
left" are different results, and a rate-limited traversal is distinguishable from an
exhausted one.

## Getting started

Start with the specification if you want to know what a document means, and with the
engine if you want one. Both repositories build with the toolchain they pin:

```bash
git clone https://github.com/Amasario-Soroban-Click/amasario-provenance-engine
cd amasario-provenance-engine
rustup show                                  # installs the pinned toolchain and targets
cargo build --workspace --all-features
cargo test  --workspace --all-features
```

Every repository carries a `CONTRIBUTING.md` with the checks its CI runs, a `SECURITY.md`
stating what is in scope, and issue templates for the kinds of change each one accepts.
