# Adapter contract and next work

`agent-evidence-vectors` tests evidence formats and verifier behavior.
`probity-verify` accepts artifact bytes and a separate consumer policy,
then emits a decision for one claim. A producer can update a case but cannot
withdraw a policy requirement. The packet binds the policy by SHA-256.

Adapters are selected by versioned `claim_type`. Each returns
`schema_version`, `case_id`, `claim_type`, `decision`, `reason`, `scope`,
`checks`, and `policy_sha256`. Unknown versions fail. Decisions are
`supported`, `contradicted`, or `not_established`.

The first adapter, `source_text_coverage/v1`, checks consumer-selected
passages against pinned source and report bytes. A policy's authority
string is a statement by the consumer, not proof of provenance.

| Field case | Candidate adapter | Required witness |
| --- | --- | --- |
| AIID report 15 | `source_text_coverage/v1` | Source capture with returned time; bound report version. An April intake cannot be judged from a June capture. |
| AVE issue 298 / PR 299 | `ave_vantage/v1` | Methodology, engine set, and an external authority probe or substrate observation for claims that require one. |
| FINOS AI governance 359 | `operand_lineage/v1` | Source values, units, time ranges, and recomputable transformations. |
| CoSAI 189 | `sufficiency_boundary/v1` | Consumer-required assessment classes and observed coverage. |
| Agent execution / in-toto AIA | `execution_observation/v1` | Consumer-pinned substrate key and coverage policy, with AEE verifier output as an input. |

Source coverage, exact operand lineage, and a pinned authority capture are implemented.
The next steps are:

1. Obtain the actual AIID report export and source capture. Keep the April
   intake `not_established` unless an April artifact is available.
2. Add source-coverage vectors to `agent-evidence-vectors` and run this
   verifier as an external implementation. Include omitted passages,
   tampered bytes, hidden script text, wrong capture time, and withdrawn
   requirements.
3. Bind a real external authority capture for an AVE finding. The class stamp
   alone cannot establish the finding's observation.
4. Bind an independently captured execution trace and source values for a
   FINOS field case. Build a separate sufficiency adapter for CoSAI.
5. The local decision index now retains replayable packets, challenges, and
   superseding decisions without replacing old entries. Next, bind claims to
   independently identified witnesses and publish a verifiable history that
   another operator cannot replace wholesale.
