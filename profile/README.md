# Amasario

**Soroban contract provenance, dependency and impact infrastructure.**

[![Explorer](https://img.shields.io/badge/explorer-live-000000?logo=vercel)](https://amasario-explorer.vercel.app)
[![Walkthrough](https://img.shields.io/badge/%E2%96%B6_watch-the_5--minute_walkthrough-58a6ff)](https://amasario-explorer.vercel.app/pitch/amasario-pitch-v2.mp4)
[![Engine CI](https://github.com/Amasario-Soroban-Click/amasario-provenance-engine/actions/workflows/ci.yml/badge.svg)](https://github.com/Amasario-Soroban-Click/amasario-provenance-engine/actions/workflows/ci.yml)
[![Reference build](https://github.com/Amasario-Soroban-Click/amasario-provenance-engine/actions/workflows/reference.yml/badge.svg)](https://github.com/Amasario-Soroban-Click/amasario-provenance-engine/actions/workflows/reference.yml)

[![Press play: the five-minute walkthrough](https://amasario-explorer.vercel.app/pitch/amasario-pitch-thumbnail.png)](https://amasario-explorer.vercel.app/pitch/amasario-pitch-v2.mp4)

Amasario answers one question about a Soroban contract, and it answers it about a chain
rather than about a repository:

> What does this contract depend on, where did its deployed artifact come from, what
> evidence supports those relationships, and what else could be affected by a change?

The answers are documents: JSON, Markdown, DOT, GraphML and JUnit, each of which states
what was observed, what was inferred, what was verified, what is unknown and what failed
— as separate things that never merge.

[**amasario-explorer.vercel.app**](https://amasario-explorer.vercel.app) is the deployment,
and the shortest way to see what the documents look like. It renders the engine's own
committed documents — the dependency graphs, the snapshot pair, the reference contract's
decoded modules — with the digest of every vendored file re-checked in its own CI. It
performs no analysis: it is a viewer, deliberately, so that a diagram cannot claim more
than the table beside it.

## The layers

Four repositories carry the layers, and the boundaries between them are the design rather
than an accident of packaging. This profile is a fifth repository in the organisation, and
it holds no layer: it is the front door to the other four.

| Repository | Layer | What it is |
| --- | --- | --- |
| [`amasario-provenance-spec`](https://github.com/Amasario-Soroban-Click/amasario-provenance-spec) | **normative** | What Amasario means by contract identity, artifact identity, provenance, dependency, evidence, confidence and impact: 24 strict JSON Schemas, 10 taxonomies, 24 models, 18 rules, 17 fixtures, 13 deterministic vectors. It performs no analysis. |
| [`amasario-provenance-engine`](https://github.com/Amasario-Soroban-Click/amasario-provenance-engine) | **execution** | The Rust CLI and libraries that consume the specification: contract inspection, network observation, evidence collection, provenance verification, dependency discovery, graph construction, impact analysis, snapshots and reports. Read-only, with no command that submits a transaction and no key handling anywhere. |
| [`amasario-explorer`](https://github.com/Amasario-Soroban-Click/amasario-explorer) | **presentation** | A browser over the engine's own documents — dependency graphs, provenance chains, snapshot diffs, verification scope and the reference contract's decoded interface. Live at [amasario-explorer.vercel.app](https://amasario-explorer.vercel.app). |
| [`amasario-docs`](https://github.com/Amasario-Soroban-Click/amasario-docs) | **cross-cutting** | What belongs to no single layer: how the layers fit, what each refuses to claim, the compatibility policy, the governance, and the gaps that are known and not yet closed. |

Two of those layers are also reachable from outside the repositories. The explorer is a live
deployment, and the reference contract pair under the engine's
[`reference-contract/`](https://github.com/Amasario-Soroban-Click/amasario-provenance-engine/tree/main/reference-contract)
is deployed to Testnet — callee
[`CBMPDHYWBGBJ4JAUKNLE6OTC4LQTLV3XFVMAN72MCFSMN2EOJPYEXK6N`](https://stellar.expert/explorer/testnet/contract/CBMPDHYWBGBJ4JAUKNLE6OTC4LQTLV3XFVMAN72MCFSMN2EOJPYEXK6N),
caller
[`CBNCEDVA7SQ2NSNGG7RGQOK4VESBN2YSCLJ6DSHRL6QH72VPR5MYIVCA`](https://stellar.expert/explorer/testnet/contract/CBNCEDVA7SQ2NSNGG7RGQOK4VESBN2YSCLJ6DSHRL6QH72VPR5MYIVCA) —
with both deployed modules hashing to the committed fixtures, so "the module this project
builds" and "the module that is running" are the same bytes and it is the chain that says
so. It is a fixture rather than a service: nothing calls it on a schedule, and it exists so
that the dependency and provenance analysis has a target this project owns rather than only
one it borrows. The engine still holds no key and signs nothing; the deployment is made by a
committed script that names an identity the `stellar` CLI keeps in its own keystore.

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

Each repository carries a `CONTRIBUTING.md` naming the checks its CI runs and a `SECURITY.md`
stating what is in scope. The engine's are the longest because it has the most surface; the
two newer repositories state the same rules more briefly, which is a difference in size rather
than in standard. If the change you want to make has no issue for it, that is usually a gap in
the issue list rather than in your idea — opening one is a contribution in itself.
