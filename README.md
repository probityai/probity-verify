# probity-verify

An offline verifier for claims against consumer-pinned evidence. The first
adapter checks whether a stored report contains passages selected from a
separate source capture.

## Run

Python 3.10 or newer and [uv](https://docs.astral.sh/uv/):

```sh
uv sync --extra test
uv run pytest
uv run probity-verify examples/source-coverage/case.json \
  --policy examples/source-coverage/policy-june.json
```

Add `--json` for the full decision or `--packet decision.txt` to save
the text packet. The synthetic June case returns
`contradicted`: a selected passage appears in the source capture but not in
the stored report. Run with `policy-april.json` to get `not_established`:
the June capture cannot establish an April intake. `case-complete.json`
with `policy-complete.json` returns `supported`.

## Inputs and decisions

The case supplies local artifacts with relative paths, byte lengths, and
SHA-256 digests. A separate consumer policy selects the witness, pins its
digest and capture time, pins the report digest and version, sets a UTC
source window, and lists required literal passages. The case cannot change
these requirements.

| Decision | Meaning |
| --- | --- |
| `supported` | Every selected passage occurs in the pinned source and bound report. |
| `contradicted` | A selected passage occurs in the source but is missing from the report. |
| `not_established` | A binding, pin, time window, or source passage check is unavailable or fails to establish the comparison. |

The verifier checks supplied bytes; it does not fetch URLs or authenticate
the authority named in a policy. The consumer must inspect and pin the
capture independently. HTML extraction reads `<article>` text and omits
scripts, styles, and templates. It does not evaluate CSS or infer what a
browser displayed. Text matching does not establish that the selected
passages are material or that the report is otherwise complete.

Malformed input exits 2. A valid decision exits 0. The packet includes the
policy digest, artifact bindings, checks, reason, and scope.

See the [adapter plan](docs/architecture.md) and
[AIID field note](examples/source-coverage/AIID-FIELD-NOTE.md).
The executable fixtures contain invented text.
