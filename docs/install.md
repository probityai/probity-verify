# Install a pinned version and check a claim

Probity Verify checks a claim against local evidence and a consumer-held policy.
Use it when you need a decision you can replay from the same bytes.
Python 3.10 or newer and [uv](https://docs.astral.sh/uv/) are required.

Version 0.1.0 is source-only and unreleased. Install from an exact public commit.
The sample artifacts come from that same commit; the wheel does not contain them.
This recipe pins a qualified source snapshot rather than a moving branch.

## Install outside the source checkout

Run these commands from a new working directory:

```sh
git clone https://github.com/probityai/probity-verify.git verify-source
git -C verify-source checkout --detach 4eb2bac542f784b68c524b99cac9eccb32b0e5cc
uv venv --python 3.12 verify-env
uv pip install --python verify-env/bin/python ./verify-source
verify-env/bin/probity-verify verify-source/examples/source-coverage/case.json \
  --policy verify-source/examples/source-coverage/policy-june.json
```

Python 3.12 is the recipe's tested runtime. Choose another supported Python
version with `--python` when you create the environment.
The install builds a wheel. The command uses that installed package, without
`uv run`, an editable install or the source checkout's Python path.

The packet includes:

```text
Case: "source-coverage-demo"
Claim: source_text_coverage/v1
Decision: contradicted
Reason: selected_span_missing_from_record
```

The retained source contains a selected passage that the report omits.
This decision concerns that passage and the supplied policy, not the entire report.

## Replay the other outcomes

Add `--json` for a structured decision:

```sh
verify-env/bin/probity-verify verify-source/examples/source-coverage/case.json \
  --policy verify-source/examples/source-coverage/policy-april.json --json
verify-env/bin/probity-verify verify-source/examples/source-coverage/case-complete.json \
  --policy verify-source/examples/source-coverage/policy-complete.json --json
```

The April policy returns `not_established`. A June capture cannot establish
what the source contained in April. The complete report returns `supported`.
All three valid decisions exit zero. A contradiction is a valid decision,
so a CI consumer must inspect the `decision` field if it needs a policy gate.

## Errors and scope

A missing file, invalid JSON or invalid case exits two and prints an error
to standard error without a verdict. An artifact digest mismatch refuses the
case; do not edit the pinned sample bytes to repair it. Restore the sample
from the pinned commit instead.

Run from the directory that contains `verify-env` and `verify-source`.
The verifier resolves artifact paths from the case's directory by default.
Use `--artifact-root` only when you deliberately keep artifacts elsewhere.

The verifier checks local bytes. It does not fetch URLs or authenticate the
authority named by a policy. The consumer must choose and pin that authority.
See [Running the verifier](running.md) for input fields and adapter examples,
and [Replayable decisions](bundles-and-index.md) to share a checked packet.
