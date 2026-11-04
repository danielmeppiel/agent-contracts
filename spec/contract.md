# Agent Contract

An Agent Contract defines one unit of agent work before it runs. This document
describes the form APMX 0.8.0 accepts and marks proposals.

## File form

A contract is a UTF-8 file named `<name>.contract.md`: YAML front matter
between two `---` lines, then a Markdown body. The body MUST NOT be empty.

[schemas/agent-contract.schema.json](schemas/agent-contract.schema.json)
describes the front matter; [examples/build.contract.md](examples/build.contract.md)
is a complete real contract.

## Fields and the six parts

The parts fall into three groups: the input (assignment), the job (role,
dependencies, budget), and how the work is judged and handed back (checks,
deliverables).

| Part | Where it lives | Rule in APMX 0.8.0 |
| --- | --- | --- |
| **Assignment** | A `needs` file that no contract in the factory produces, supplied per run (in the demo factory, `request.md`) | Recorded by digest as a run input; not part of the contract's identity |
| **Role** | The Markdown body | Non-empty; given to the harness as the task |
| **Dependencies** | `imports`: names of skills declared in the package manifest | Names only; versions are pinned by the package lock (APM's `apm.lock.yaml`) |
| **Budget** | `budget: {max_attempts, max_seconds}` | Both or neither; one attempt without a budget |
| **Checks** | `verify`: map of check name to command | At least one |
| **Deliverables** | `produces`: one path or a list | At least one; paths must not collide or overlap |

`needs` is one path or a list (possibly empty). A `needs` path that another
contract produces is a **handoff**: the consumer receives that exact checked
file. Paths are fixed relative paths with no globs, absolute paths, `.` or `..`
segments, or `.git`, `.apm` or `apm_modules` segments. `needs` is not a source
inventory; APMX captures the project workspace separately.

**Implementation limits.** APMX 0.8.0 caps `needs` at 16 paths, `produces` at
16, `verify` at 8 checks (names matching `[A-Za-z0-9][A-Za-z0-9_-]{0,63}`),
`imports` at 256, `max_attempts` at 1–16 and `max_seconds` at (0, 86400]. The
schema encodes these. Whether they belong in the format is open.

**Check semantics.** A check runs after the agent finishes, outside its
conversation, over the captured delivered files. Exit 0 passes; exit 1 is a
rejection, retried while budget remains; any other exit is an operational
failure that stops the run. Every declared output must exist and every check
pass before a downstream contract receives anything. Commands are
uninterpreted strings; this draft defines no command language.

**Two layers of checks.** The contract holds generic, reusable checks for a
kind of work (in the demo: acceptance runner, regression suite, change scope,
money in cents, report format). Task-specific acceptance scenarios live in the
target repository (`acceptance/`), written before the run; the acceptance
check reads them from the captured original files, and a patch that touches
them is rejected. The split mirrors a CI pipeline and the tests in a
repository.

**Dependencies are not checks.** `imports` says which skills the agent is
given; the Receipt records them, pinned, so a gate can enforce that only
approved skills were supplied. Whether the agent followed them is not
recorded; that is what the checks are for.

**Reserved and unknown keys.** APMX rejects any other front-matter key. It
reserves `run` and `sandbox` and refuses contracts that use them.

## Identity

- **Contract digest**: the SHA-256 of the file's exact bytes
  (`contractSha256` in the Receipt). For the example contract it is
  `0a3c8ad0d6a2ebf135c27e6a43884f40442c4e9c8e95aeca9d7627086981d4f6`.
- **Check resources**: commands reference files, so APMX also records every
  file under the factory's `checks/` directory by SHA-256 and size.
- **Factory fingerprint**: the SHA-256 of the canonical factory definition
  (JSON with sorted keys, compact separators, ASCII escaping and a final LF;
  APMX schema `apmx-definition/1`). Per stage it covers the contract digest,
  `needs`, `produces`, checks, check resources, selected skill content and pins,
  limits, budget and the workspace selection profile. It also covers the
  resolved package dependencies, which include the factory package's own commit
  and content hash when it runs from a package. It excludes harness, model,
  clocks and machine paths.

So one factory has one fingerprint across harnesses: the APMX demo's Copilot
CLI and OpenCode Receipts for `apmx-demo-factory#v2.0.0` both carry `f6c2d240…`.

*Proposed:* a standalone **contract identity**, defined as the digest of the
contract's stage entry in the definition. APMX computes these entries only
inside a factory definition today.

## Composition into a factory

A factory is a directory of contracts with their check resources and a package
manifest. There is no pipeline file; the graph is derived from paths:

- An edge runs from A to B when B `needs` a path A `produces`.
- Each path MUST be produced by at most one contract.
- The graph MUST be acyclic.
- Produced paths MUST NOT overlap each other, caller inputs, contract sources
  or managed resources (`checks/`, manifest, lock).
- `needs` paths that no contract produces are caller inputs: the assignment.

APMX also caps a factory at 64 contracts. The demo factory chains five:
planning, specification, build, documentation and review, each reading
`request.md` and earlier outputs.

A factory is versioned as a package: APMX resolves it through APM (`owner/repo#ref`), and an organization approves it by
fingerprint, not by tag (see [gate.md](gate.md)).

## Portability

The aim is that one contract runs unchanged on more than one harness. Rules
for the core format:

- Front matter MUST NOT name a harness or model.
- The body SHOULD describe the work and deliverables, not a harness's
  interface.

This is not yet enough. The example contract's body names `write_file` and
`export_changes`, tools APMX supplies to the harness. Such a contract is
portable only across harnesses that receive them.

*Proposed:* optional **harness bindings** as namespaced extension keys (for
example `x-<binding>`) that core implementations ignore. This conflicts with
APMX's current rejection of unknown keys and needs a change there too.
