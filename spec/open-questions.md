# Open questions

Questions we cannot answer alone. Please raise them in
[Discussions](https://github.com/danielmeppiel/agent-contracts/discussions).

## Model attestation

Receipts record the models a harness reports; nothing attests them, and one
demo harness reported none. Should a model provider or harness sign which model
served a session, and how would a Receipt reference it?

## Harness portability bindings

Contract bodies can depend on tools the runtime supplies. How should a contract
declare that, so a harness can refuse what it cannot honour? What must every
harness provide (file capture, tool surface, model reporting)?

## SLSA field placement and neutral identifiers

Where should caller choices (harness, requested model, assignment) and
observations (attempts, reported models, stage outputs) live in SLSA
Provenance v1? Should the re-checking job be the builder, and the producer a
separate statement? What neutral `buildType`, annotation key, profile
identifier and media type should replace the APMX URIs, and should the profile
be proposed to the in-toto attestation predicate registry?

## Subjects of check results

Every statement names the delivered files, so lookup by file digest works,
even when a check examined `plan.md` or a patch. Should a check result also
name, or instead name, the artifact it tested?

## ABOM profile alignment with CycloneDX

Today's components are packages with a purl, no `hashes`, and no skill-level
entries; skill digests live in the provenance. Should skills be components,
with which `type`, purl scheme and hash? Should MCP servers and tools appear?
Is CycloneDX 1.6 or later, or SPDX 3.0's AI profile, a better home? We would
like to work with the CycloneDX community on this.

## Revocation identity

APMX revokes on purl plus skill-body digest, but the purl names a package
commit (and `pkg:generic` drops the owner). What identity should revoke one
skill, and should it cover the skill's other resources?

## Organization-wide receipt search

"Which merged changes used this skill version?" Today that is scripting over
the attestation store, one digest at a time. What index or query should
attestation stores offer?

## Check equivalence and check inputs

A check is a command plus the files under `checks/`; which files a command
reads is not declared. When are two checks in different ecosystems the same
check? Should checks declare what they assess and read, and can a verifier
trust that? How should flaky or non-deterministic checks be treated when
re-checking?

## Isolating checks

Checks can run agent-written code inside the job that writes the Receipt of
record. What isolation should a re-checking job provide, and should the
Receipt record it?

## Signing beyond one CI platform

APMX signs with GitHub Actions OIDC and Sigstore's public-good instance and
verifies with `gh attestation verify`. How should signer identity be expressed
for reusable workflows versus their callers, for other CI providers, and for
offline or private deployments? How long is a verification summary valid, and
how is replay of an old one handled?

## Several Receipts per change

A pull request may combine deliveries from several runs or factories. Should a
gate accept a union of summaries, and how are path prefixes between workspace
and repository specified?

## Confidentiality

Provenance inlines the package manifest and lock, and names transcripts and
inputs by digest. What may appear in a public attestation store?

## Contract versioning semantics

Contracts have digests but no version field or compatibility rule. Is a body
change breaking? A check change? How should a format version appear in the
file without breaking 0.1 front matter?

## Relation to other emerging work

How should Agent Contracts relate to [Agent Plugins](https://agent-plugins.org/)
(packaging skills and tool configuration), agent instruction files and agent
workflow formats? Where should this format stop so it composes with them?
