# What did not work

The prompt has one failure I could reproduce, and it is the most useful thing in this repo.

## Directive text inside a memory record does not enforce itself

After the cost record was superseded (see [blobs.md](blobs.md)), the corrected record contained a
literal instruction:

> *Never state the cost story as "a small multiple of the blob size" without the fixed-overhead half.*

While drafting the 1,290-word explainer, working from that exact record, the agent wrote an
unqualified cost claim anyway. One question surfaced it and the correction was accurate and
complete — but the guard did not fire on its own.

**The scoreboard from the cross-client demo:**

| test | result |
|---|---|
| Blatant contradiction — asked to assert full replication | **Caught.** Refused, cited the mechanism |
| Subtle contradiction — an unqualified cost claim | **Missed on first pass**, while drafting from the record that forbade it |
| Same, on challenge | **Caught.** Correction accurate |

## Why it happens

A memory record is context. An instruction inside context competes with everything else in the
window, and it loses to whatever reads more fluently. The contradiction-guard compares a draft
against *facts*; it was never built to execute *directives* it finds inside them.

## What follows from it

Rules belong in the prompt. Records hold facts. If a constraint matters, it goes in §2 where the
guard runs, not into the body of a canon record where it reads as prose.

This is recorded in `walrus-notes::chronicle` as a finding, so the next session inherits it rather
than rediscovering it.
