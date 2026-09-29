# probity-verify

An offline verifier for claims against consumer-pinned evidence. Its adapters
check selected source passages, exact operand lineage, and a field in a bound
external authority capture.

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

## Share a replayable decision

```sh
uv run probity-bundle create examples/source-coverage/case.json \
  --policy examples/source-coverage/policy-june.json --output decision.zip
uv run probity-bundle replay decision.zip
```

The ZIP contains the case, consumer policy, bound artifact bytes, and decision.
Replay checks the bindings, recomputes the decision, and rejects unsafe archive
members. Only fully bound cases can be bundled. Identical inputs produce
identical ZIP bytes. The ZIP is unsigned; its holder can replace the policy,
artifacts, and decision together. A recipient must establish the policy and
witness provenance separately.

Malformed input exits 2. A valid decision exits 0. The packet includes the
policy digest, artifact bindings, checks, reason, and scope.

See the [adapter plan](docs/architecture.md) and
[AIID field note](examples/source-coverage/AIID-FIELD-NOTE.md).
The executable fixtures contain invented text.

The [source coverage vectors](vectors-source-coverage/README.md) test six
boundary cases with pinned fixture bytes, expected decisions, and reasons.
Run them with `uv run --extra test pytest vectors-source-coverage/tests`.

The [operand-lineage vectors](vectors-operand-lineage/README.md) test ten
arithmetic and evidence-boundary cases against an external CLI.

## Operand lineage

`operand_lineage/v1` recomputes ordered binary arithmetic over exact decimal
strings. A consumer policy pins separate source and execution artifacts,
declares constants, and names the final step. Every declared constant and
step must contribute to that figure. A missing or unbound artifact yields
`not_established`; missing references also yield `not_established`.
Mismatched known operands or results yield `contradicted`.
Nonterminating division needs a rounding rule and yields `not_established`.
Malformed input exits 2 without a verdict.

`supported` means the supplied trace is internally consistent with the
selected source bytes. The policy's authority label does not authenticate
who observed execution, establish source completeness, or show that the
chosen operation answered the intended question. A real execution claim
needs an independently captured trace and its own binding policy.

## Authority anchor

`authority_anchor/v1` compares a consumer-selected JSON field with an expected
identity in a pinned HTTP capture. It checks the request target, status, and
capture window. A different bound identity is `contradicted`; an absent,
unreadable, wrong-target, or stale capture is `not_established`.

The consumer must pin a capture obtained outside the observed artifact and
establish its provenance. The verifier does not contact the authority or
authenticate the transport or capture clock. An AVE class stamp naming
`external_authority` describes a possible vantage; it is not evidence that a
particular finding made that probe. The example uses invented identities.
