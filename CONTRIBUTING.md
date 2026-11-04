# Contributing

Draft 0.1 is an invitation. The most useful contributions are use cases,
counter-examples and corrections from people who run harnesses, attestation
stores, SBOM tooling or large codebases.

## How to propose a change

1. **Start a discussion** in [Discussions](https://github.com/danielmeppiel/agent-contracts/discussions)
   for anything beyond a typo: a new field, a changed rule, an answer to an
   [open question](spec/open-questions.md). Say what problem it solves and
   what it costs implementers.
2. **Open an issue** for a concrete defect: an ambiguity, an incorrect use of
   in-toto, SLSA, CycloneDX, DSSE or Sigstore, or a statement that an
   implementation does not back.
3. **Open a pull request** once a direction is agreed. Keep it small. Change
   the schema and examples in the same pull request as the text they follow.

## Rules for spec text

- Separate what an implementation does today from what is proposed.
- Use RFC 2119 keywords only for real requirements.
- Prefer reusing an existing standard field over inventing one; when no
  standard home exists, say so.
- No claims that code is correct or secure, and no SLSA levels. A Receipt is trusted
  evidence about how files were produced and checked, nothing more.

## Sign-off

Sign off each commit to certify the
[Developer Certificate of Origin](https://developercertificate.org/):

```sh
git commit -s
```

## Licences

Contributions to specification text are licensed under CC-BY-4.0
([LICENSE-docs](LICENSE-docs)); contributions to schemas, examples and code
under Apache-2.0 ([LICENSE](LICENSE)).
