# Reduce review-lens detection samples 2 → 1 (keep security at 2)

The review panel is expensive enough that two runs exhaust a session limit. On
#605 (L-size PR) the panel cost **~$14 / 428 turns / 73 min for one round**, its
verifier sessions hit session-limit 429s (6 of 10 errored → findings unfiltered),
and the docs lens blew its turn ceiling — the loop paged (`agent:blocked`). The
per-lens **token attribution** (from #603) showed the sink is **detection, not the
verifier**: $11.60 detection vs $2.74 verifier, dominated by `correctness` $3.81,
`blast-radius` $2.97, `design-fit` $2.29 — each at `samples: 2` (two opus runs).

## The change

- [x] `scripts/agent/lenses/lenses.json`: all blocking lenses → `samples: 1`
      (`correctness`, `security`, `design-fit`, `test-adequacy`, `blast-radius`;
      `docs` was already 1).
- [x] Design-doc note (`harness-engineering.md`, sampling section) explaining the
      cut and its rationale.

## Why

`samples: 2` + union exists to raise recall against single-sample non-determinism
(the #521 false negative). Reducing to 1 trades that recall for ~halved detection
cost — the panel's single largest expense (#605). The trade is safest for the
code-quality lenses, where a missed defect is still backstopped by the **verifier**,
the **tests**, and **CI**.

`security` was initially kept at 2 for exactly one reason — a planted instruction
or pasted secret it misses has *no* backstop — but was **also dropped to 1** on
request, accepting that sharper edge for the cost. The knob is per-lens, so raising
it back is a one-field change if injection/secret misses show up.

Secondary benefit: fewer raw findings per round → less load on the verifier,
which is what was hitting 429s.

## Not done here

- The docs lens `error_max_turns` page (the proximate #605 trigger) is a separate
  fix — either raise `docs.maxTurns` or stop a detection turn-limit from failing
  the whole lens closed as a blocking no-verdict.
- Whether to also drop `correctness`/`security` further is left for the attribution
  table to inform; the knob is per-lens.
