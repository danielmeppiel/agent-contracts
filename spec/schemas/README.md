# Schemas

| File | Describes |
| --- | --- |
| [agent-contract.schema.json](agent-contract.schema.json) | The YAML front matter of an Agent Contract (JSON Schema 2020-12) |

The schema follows what APMX 0.8.0 accepts, including its implementation
limits. It does not express: case-insensitive duplicates and reserved
segments, overlapping `produces` paths, the 256 KiB source limit, NUL bytes,
the non-empty Markdown body, or graph rules (one producer per path, no
cycles). Implementations enforce those. Parse the front matter as
YAML, then validate the resulting object.

The Receipt has no schema of its own: each statement validates against its
upstream format (in-toto Statement v1, SLSA Provenance v1, in-toto Test Result
v0.1, CycloneDX 1.5, SLSA VSA v1). A schema for the profile's constraints, such
as identical subjects across statements, is future work.

Licence: Apache-2.0 (see [LICENSE](../../LICENSE)).
