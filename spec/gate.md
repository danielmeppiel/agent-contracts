# Gate

A gate is one required check at merge that reads Receipts against the
organization's rules. APMX's `apmx audit` is mapped at the end.

## Rules

Rules are an organization's policy for factory deliveries. In APMX they live
in `apm-policy.yml`: APM's own keys (such as `dependencies.allow` and `deny`)
govern skill and package sources, and a `factories` section, read by APMX,
governs deliveries. The receiving demo repository's section:

```yaml
factories:
  allow:            # approved fingerprints; the package and tag are a label
    - "danielmeppiel/apmx-demo-factory#v2.0.0@sha256:f6c2d240…"
  require_signed: true
  signers:          # trusted signers, pinned by commit
    - workflow: danielmeppiel/apmx/.github/workflows/run.yml@refs/tags/v0.8.0
      workflow_sha: "fae2067a0dc6ff07beda714015ff9e648f41ce63"
      repo: danielmeppiel/contoso-payments
  revoked: []       # skills revoked by exact identity and content digest
  enforcement: block
```

Rules compose across organization and repository levels **tighten-only**: a
lower level can narrow allow-lists or add revocations, never relax them.

## What a gate SHOULD verify

1. **Rules come from the protected base.** The gate SHOULD read the rules, and
   run its own definition, from the target branch, never from the change under
   review, so a change cannot loosen its own gate.
2. **Trusted signer.** A gate SHOULD require a verification summary signed by
   a signer the rules name, pinned to the signer's code by commit digest, not
   by a mutable tag or branch.
3. **Coverage.** Every file the change adds or modifies SHOULD be a subject of
   one passing summary from an allowed signer, with the same digest; a
   delivery plus unrelated edits fails. Deletions SHOULD match the deletions
   in the provenance (they are not subjects).
4. **Bound statements.** The gate SHOULD keep only the statements the summary
   names by digest (`inputAttestations`) and check those: well-formed against
   their schemas, same subjects, one passing check result per declared check.
5. **Approved factory.** The provenance's factory fingerprint SHOULD be on the
   allow-list. A package name or tag alone SHOULD NOT approve a factory.
6. **Dependency sources.** The recorded package lock SHOULD satisfy the
   organization's dependency rules (allowed sources, denials). APMX delegates
   this to APM by reinstalling the recorded lock and running `apm audit`.
7. **Revocations.** The gate SHOULD reject a Receipt in which any skill
   matches a revocation entry, on exact identity **and** content digest, so
   an identity alone never revokes.
8. **Freshness.** Each touched file's recorded before-image SHOULD match the
   target branch (or be absent for a created file); a recorded base commit
   SHOULD be in the target's history. Otherwise the delivery is stale and the
   factory should run again.
9. **Fail closed.** A policy that cannot be loaded, or a lookup, network or
   verification error, MUST NOT be treated as a pass. It SHOULD be reported
   distinctly from a rejection.

Changes that are not factory deliveries fail the gate by design and go
through the repository's normal review path.

## Honest limits

- **Green is narrow.** It means: this factory built these files, the rules
  approved that factory, the files are unchanged since the run, a trusted
  signer vouched for the Receipt, and the factory's checks passed. It does
  not mean the code is correct; green is only as good as your checks.
- **Models are recorded, not enforced.** There is no rule on models in 0.1.
- **Revocation blocks new deliveries.** It does not recall merged code. APMX
  has no search for past deliveries that used a revoked skill; today that
  means scripting lookups over the attestation store.
- **Checks can run agent-written code** inside the job that writes the
  Receipt of record (see [receipt.md](receipt.md#verification-summary-and-signing)).
- **What you skip and what still runs.** A gate can let you skip rerunning
  the factory's own checks and a line-by-line human read of the delivery. It does
  not replace integration tests, other required workflows or a merge queue.
- **Whoever controls the target branch controls the gate.** Organization-level
  required workflows reduce, not remove, this.
- **The signer is trusted, not verified.** The gate trusts the signing
  workflow's code (pinned by commit), the Sigstore instance and the CI
  platform's identity issuer.

## Mapping: `apmx audit` (APMX 0.8.0)

| `apmx audit` row | Gate items |
| --- | --- |
| Policy | 1 (read from the PR base; unloadable is INCOMPLETE, exit 2) |
| Integrity | 4 (canonical envelopes, line digests, schemas) |
| Factory | 5 (`factories.allow` by fingerprint) |
| Checks | 4 (one PASSED result per declared check) |
| Dependencies | 6 (`apm audit --ci` on the reinstalled lock) |
| Revocation | 7 (`purl` and `bodySha256` both match) |
| Outputs | 3, 8 (subjects, before-images, creations, deletions; stale base) |
| Identity | 2, 3 (signed VSA from `factories.signers`, `workflow_sha` pin) |

Exit codes: 0 VALID, 1 INVALID (including "not a factory delivery"),
2 INCOMPLETE (could not decide; never a pass). It accepts `pull_request` and
`pull_request_target` events; APMX recommends `pull_request_target`, so the
base branch's copy of the gate runs.
