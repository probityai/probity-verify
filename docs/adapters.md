# Adapters

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

## Event absence

`event_absence/v1` checks a claim that no event of a selected type occurred
for one invocation and interval. A bound event in that interval contradicts
the claim even if coverage is incomplete. An empty record supports it only
when a separate capability record declares field visibility and the
observation record declares complete coverage for that claim and scope. The
same rule applies when the event field is absent or present but empty.

Missing visibility or coverage is `not_established`. Malformed records fail
without a verdict. Both records are pinned by the consumer policy. Matching
identifiers and digests do not authenticate the producer or prove that its
coverage statement is true; the consumer must establish those facts separately.

An observation with `coverage: incomplete` must name its gaps as `start` and
`end` intervals inside its scope. Use `coverage: unknown` when no gap can be
located. A bare `incomplete` flag is malformed, not a property verdict.
