# Examples

Both examples are real, not normative.

| File | What it is |
| --- | --- |
| [build.contract.md](build.contract.md) | The build contract of [`danielmeppiel/apmx-demo-factory`](https://github.com/danielmeppiel/apmx-demo-factory) at tag `v2.0.0` (commit `962df453559fc894915271469cf83fe79cd0dcb9`), byte for byte. Its SHA-256 is `0a3c8ad0d6a2ebf135c27e6a43884f40442c4e9c8e95aeca9d7627086981d4f6`, the `contractSha256` of the build stage in the Receipt excerpt. |
| [receipt-excerpt.json](receipt-excerpt.json) | A decoded, trimmed Receipt written by APMX 0.8.0 for a local, unsigned run of that factory with the Copilot CLI harness. Digests are truncated and repeated elements omitted (marked `…`), so it does not validate as-is. Strings are verbatim APMX output, including its `scope` annotation. |

The demo factory is licensed under Apache-2.0; the contract is
reproduced unchanged under that licence. Its body names `write_file` and
`export_changes`, tools APMX supplies to the harness (see
[contract.md](../contract.md#portability)).

Check the digest yourself:

```sh
shasum -a 256 spec/examples/build.contract.md
```
