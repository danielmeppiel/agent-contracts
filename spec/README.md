# Agent Contracts & Receipts — format draft 0.1

**Status:** draft 0.1, November 2026. An invitation to shape a format, not a
standard. Nothing here is stable. Text: CC-BY-4.0. Schemas and examples:
Apache-2.0.

## Overview

- An **Agent Contract** ([contract.md](contract.md)) defines one unit of agent
  work before it runs.
- A **Receipt** ([receipt.md](receipt.md)) is the evidence one run leaves about
  the files it delivered: a profile over in-toto, SLSA and CycloneDX.
- A **gate** ([gate.md](gate.md)) reads Receipts against an organization's
  rules at merge.

Everything here is generalised from one open-source implementation,
[APMX](https://github.com/danielmeppiel/apmx) 0.8.0. Each document separates
**what APMX does today** from **what this draft proposes**. Where we lack an
answer, see [open-questions.md](open-questions.md). A real contract and a real,
trimmed Receipt are in [examples/](examples/); the front-matter schema is in
[schemas/](schemas/).

## Terminology

- **Agent Contract**: a versioned, identifiable definition of one unit of agent
  work, in six parts: assignment (the intent supplied for each run), role
  (instructions), dependencies (packaged skills), budget (tries and time),
  checks and deliverables.
- **Agentic software factory**: a graph of Agent Contracts connected by files,
  packaged and versioned, run by a harness.
- **Factory fingerprint**: the digest of the factory definition, which pins
  every contract, check resource and skill.
- **Harness**: the coding agent that runs a contract (in APMX: Copilot CLI or
  OpenCode).
- **Check**: a named command, fixed in the contract before the run, that judges
  delivered files outside the agent's conversation.
- **Receipt**: one in-toto bundle about the delivered files: provenance, check
  results and an agent bill of materials (**ABOM**, a CycloneDX inventory of
  the packages supplied to the agents).
- **Trusted signer**: a verifier outside the producing agent's reach, named by
  policy, that signs a summary of the Receipt.
- **Rules**: an organization's policy for deliveries (in APMX: `apm-policy.yml`).
- **Gate**: one required check at merge that reads Receipts against the rules.

## Conformance language

The key words MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are to be interpreted
as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) and
[RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when, they
appear in all capitals. Most of this draft is descriptive. It defines no
conformance classes or test suite.

## Scope

The file form and fields of an Agent Contract; how contracts compose into a
factory and how a factory is identified; which statements a Receipt contains
and which fields carry which facts; what a gate should verify.

## Non-goals

- **Not a runtime or a sandbox.** How a harness executes a contract, and how
  processes, network and credentials are isolated, belong to the runtime or CI
  platform.
- **Not a model policy.** Receipts record the models a harness reports. They do
  not enforce or attest them.
- **Not a correctness claim.** What a passing gate means is stated once, in
  [gate.md](gate.md#honest-limits). It is as good as the checks.
- **No SLSA level is claimed.** The Receipt uses SLSA formats; it does not meet
  or assert any SLSA Build level.
- **Not a replacement for in-toto, SLSA or CycloneDX.** It is a profile: which
  predicates appear and which facts go in which field. Where a fact has no
  clear standard home, it says so.
- **Not a packaging format.** Contracts and skills ship through existing
  package managers (APMX uses APM, Microsoft's open-source Agent Package
  Manager).

## Prior work

- **Design by Contract** (Bertrand Meyer, Eiffel): preconditions and postconditions. In this format, `needs` and the declared checks play those roles.
- **Contract Net Protocol** (Reid G. Smith, 1980): contracts as a way to allocate tasks among agents.
- **Agent Contracts: A Formal Framework for Resource-Bounded Autonomous AI Systems** (Qing Ye and Jing Tan, 2026, [arXiv:2601.08815](https://arxiv.org/abs/2601.08815)): input/output specifications, resource budgets, temporal bounds and success criteria for agents.

This draft builds on those ideas and adds what a software supply chain needs: contracts composed into a versioned factory with one fingerprint, a receipt of existing open-standard evidence (SLSA, in-toto, CycloneDX) for every run, a signature from CI rather than from the agent, and one gate at merge that reads the receipt against policy.

## Versioning of this format

The format version is `0.1`. Until 1.0, any version MAY change incompatibly;
changes are listed in the repository history. Neither artifact carries a
format version in 0.1 (see [open questions](open-questions.md#contract-versioning-semantics)).
Embedded statements keep their own versions: in-toto Statement v1, SLSA
Provenance v1, in-toto Test Result v0.1, CycloneDX 1.5, SLSA VSA v1.
