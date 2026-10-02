# Running the verifier

## Run

Python 3.10 or newer and [uv](https://docs.astral.sh/uv/):

```sh
uv sync --extra test
uv run pytest
uv run probity-verify examples/source-coverage/case.json \
  --policy examples/source-coverage/policy-june.json
uv run probity-verify examples/operand-lineage/case.json \
  --policy examples/operand-lineage/policy.json
uv run probity-verify examples/authority-anchor/case.json \
  --policy examples/authority-anchor/policy.json
uv run probity-verify examples/event-absence/case.json \
  --policy examples/event-absence/policy.json
```

Add `--json` for the full decision or `--packet decision.txt` to save
the text packet. The synthetic June case returns
`contradicted`: a selected passage appears in the source capture but not in
the stored report. Run with `policy-april.json` to get `not_established`:
the June capture cannot establish an April intake. `case-complete.json`
with `policy-complete.json` returns `supported`.

## Inputs and decisions

The case supplies local artifacts with relative paths, byte lengths, and
SHA-256 digests. A separate consumer policy pins witnesses and sets the
claim requirements. The case cannot change them.

| Decision | Meaning |
| --- | --- |
| `supported` | The stated claim passes its adapter's checks against the pinned bytes. |
| `contradicted` | Bound evidence conflicts with a claim requirement. |
| `not_established` | Required evidence or a rule needed to decide is unavailable. |

The verifier checks supplied bytes; it does not fetch URLs or authenticate
the authority named in a policy. The consumer must inspect and pin the
capture independently. HTML extraction reads `<article>` text and omits
scripts, styles, and templates. It does not evaluate CSS or infer what a
browser displayed. Text matching does not establish that the selected
passages are material or that the report is otherwise complete.
