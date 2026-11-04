# Receipt

A Receipt is the evidence one completed run leaves about the files it
delivered. It is a profile over existing formats; it defines no new predicate.
This document describes what APMX 0.8.0 writes and marks proposals. A decoded,
trimmed real Receipt is [examples/receipt-excerpt.json](examples/receipt-excerpt.json).

Each part serves one need: provenance, **traceability** (which factory,
contract, harness and base); check results, **verification** (which checks
passed); the ABOM, **enforcement** (the mandated skills were the ones
supplied); the signed summary, **security** (untampered, vouched for by a
trusted signer).

## Bundle form

A Receipt is an [in-toto bundle](https://github.com/in-toto/attestation/blob/main/spec/v1/bundle.md):
a JSON Lines file (APMX: `receipt.intoto.jsonl`), one
[DSSE](https://github.com/secure-systems-lab/dsse) envelope per line, each with
`payloadType: application/vnd.in-toto+json` and an
[in-toto Statement v1](https://github.com/in-toto/attestation/blob/main/spec/v1/statement.md)
as payload.

| Statement | `predicateType` | Count |
| --- | --- | --- |
| Provenance | `https://slsa.dev/provenance/v1` | exactly 1 |
| Check result | `https://in-toto.io/attestation/test-result/v0.1` | 1 per (contract, check) pair of the final candidates; 11 in the example |
| ABOM | `https://cyclonedx.org/bom` (CycloneDX 1.5, inline) | exactly 1 |

**The envelopes are unsigned** (`"signatures": []`), locally and in CI, so they
are not verifiable as DSSE on their own. Authentication comes from a separate
signed verification summary that names each line by digest (below). APMX's
verifier also accepts at most one such summary as a line of the bundle; its CI
stores it separately.

Envelopes and payloads are JSON with sorted keys, compact separators and ASCII
escaping (not RFC 8785). A line's digest is the SHA-256 of its bytes without
the line feed.

APMX writes a Receipt only for a run whose every stage completed with all
checks passing; rejected or halted runs get none.

## Subject

Every statement MUST name the same subjects: the delivered files, by path and
`sha256`. When a factory delivers patches, the subjects are the files the
patches produce (applied in stage order to the starting files), not the
patches. In a local Receipt, paths are relative to the run's workspace; the
signed summary carries repository paths.

A gate looks Receipts up by these subjects. The cost: a check result names the
delivered files even when the check examined an intermediate artifact
(`plan.md`, `changes.diff`), recorded by digest in the provenance's stage
outputs ([open question](open-questions.md#subjects-of-check-results)).

## Provenance (SLSA Provenance v1)

| Fact | Field in APMX 0.8.0 |
| --- | --- |
| Build type | `buildDefinition.buildType`: `https://github.com/danielmeppiel/apmx/build/factory/v2` |
| Factory package, commit, fingerprint | `externalParameters.factory`: `package`, `commit`, `definition.sha256` |
| Per contract | `internalParameters.stages[]`: `contract`, `contractSha256`, `checks` (name, command), `budget`, `attempts`, `outputs` by digest |
| Harness | `stages[].harness`, `stages[].harnessVersion` (`<harness> --version`, or `null`) |
| Model | `stages[].requestedModel` (or `null`), `stages[].observedModels` (as reported by the harness) |
| Base | `resolvedDependencies[]`, `annotations."apmx.role": "base"`: `digest.gitTree` (in CI also `gitCommit`, `uri`), annotations `created`, `deleted`; plus one `before-image` per touched file that existed |
| Assignment and inputs | `resolvedDependencies[]`, role `input`, by digest |
| Skills | `resolvedDependencies[]`, role `capability`: `uri` (purl), `digest.sha256` of the skill body; annotations `bomRef`, `commit`, `packageHash`, `lockIdentity`, `resources` |
| Package manifests and lock | `resolvedDependencies[]`, roles `apm-manifest`, `apm-imports-manifest`, `apm-lock`, bytes inline in `content` |
| Builder | `runDetails.builder.id` (the implementation's repository), `version` |
| Definition, transcripts, rejected attempts | `runDetails.byproducts[]` by digest, roles `definition`, `transcript`, `rejected-attempt` |

Harness and model values are **recorded as reported, not verified**; the
APMX demo's OpenCode Receipt has `observedModels: []`.

**Known misfits.** SLSA reserves `internalParameters` for values the build
platform sets; harness and requested model are caller choices, and `attempts`,
`outputs`, `harnessVersion` and `observedModels` are observations, not
parameters. `created` and `deleted` describe outputs but sit on an input.
`builder.id` is a source repository, not a trust boundary. *Proposed:* caller
choices and the assignment digest in `externalParameters`, a home for
observations agreed with SLSA maintainers, and neutral identifiers in place of
the APMX URIs.

## Check results (in-toto Test Result v0.1)

| Fact | Field |
| --- | --- |
| Outcome | `result` (`PASSED` in every Receipt APMX writes) |
| Checker identity | `configuration[]`: each `checks/` file named in the command, by `name` and `digest.sha256` (the contract file if none is named) |
| Check, command, contract, fingerprint | `configuration[].annotations`: `check`, `command`, `stage`, `factoryDefinition` |
| Scope statement | `configuration[].annotations.scope` |
| Per-test names | `passedTests`, when the checker reports them |
| Raw report or log | `url`: a relative path into the optional evidence store, not a resolvable URI |

Agent output cannot be rebuilt, so it is **re-checked**: in CI, a job with no
agent (`apmx check`) reruns the factory's checks on the producer's output bytes
and writes the Receipt of record, labelling the producer's harness and models
`"reportedBy": "producer"`, `"checkedBy": "apmx check (replay)"`.

## ABOM (CycloneDX 1.5)

The agent bill of materials lists the packages supplied to the agents, from
the run's retained lock. In APMX it is APM's CycloneDX 1.5 export, inline.
Without a retained lock, APMX writes its own inventory with zero components
and says so in `metadata.properties`.

It supports enforcement, not verification: a gate can require that only
approved skills were supplied. It is not an inventory of the generated
application's dependencies or of everything that influenced the model. A
skill that was supplied may not have been followed.

**Gaps today.** Components are packages (including the factory package), with
`purl`, `bom-ref` and `version` but no `hashes`. Skills are not separate
components: each skill's content digest is in the provenance's `capability`
entry, linked by `bomRef`. The purl type follows the lock; in the example it
is `pkg:generic/agents@<commit>`, which drops the repository owner. See
[open questions](open-questions.md#abom-profile-alignment-with-cyclonedx).

## Verification summary and signing

A local Receipt is **content-bound, not authenticated**: anyone who can rewrite
the files and the Receipt can forge a self-consistent one. A Receipt used to
gate a merge MUST be authenticated by a **trusted signer**: a verifier with
credentials and a workspace the producing agent never had (in its own job, the
agent could rewrite checks or results, or sign with the job's credentials).

APMX's reusable CI workflow (`run.yml`) splits trust across three jobs:

1. **produce**: the agent, read-only repository access; its Receipt is
   untrusted data.
2. **check**: a fresh runner, no agent; reruns the checks and writes the
   Receipt of record.
3. **attest**: no checkout of the caller's code; verifies the Receipt
   offline, then signs with Sigstore (keyless, GitHub Actions OIDC).

Optional jobs fetch a target repository and open the pull request. The attest
job stores two attestations over the delivered files:

- a **SLSA Verification Summary v1**, the statement a gate trusts: `verifier.id`
  is the signing workflow at its commit; `timeVerified`; `resourceUri` is the
  base repository at its commit; `policy` is the factory fingerprint
  (`uri: apmx:factory/<hex>`, not resolvable); `inputAttestations` names each
  Receipt line by digest; `verificationResult: PASSED`; `verifiedLevels: []`;
- the bundle, in an APMX wrapper predicate
  (`https://github.com/danielmeppiel/apmx/attestation/receipt-bundle/v2`), so
  it can be fetched by file digest. The wrapper's signature asserts nothing by
  itself; GitHub's store refuses provenance with an unrecognized `buildType`.

`PASSED` means the Receipt is well-formed and the factory's own checks passed.
The organization's rules are evaluated only by the gate.

**Limit.** Checks can execute agent-written code (the demo's regression check
runs the changed application's tests) inside the job that writes the Receipt
of record; the attest job re-verifies structure, not execution. The
separation is only as strong as the check job's isolation.

## Storage and lookup

Signed attestations SHOULD be stored where they can be looked up by subject
digest. APMX uses GitHub's attestation store
(`repos/<repo>/attestations/sha256:<digest>`); the pull request carries code
only. An optional evidence store (`evidence/sha256/<hex>`) holds inputs,
transcripts, intermediate outputs and raw logs; a Receipt verifies without it.
Both it and the manifests inlined in the provenance can contain private code
or secrets.
