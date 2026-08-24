# Three measured findings

Each was measured against the live mainnet relayer, not reasoned about. Each includes the exact
call so a judge can disprove it.

---

## 1. The dedup ladder is inverted on the MCP path

**Claim.** Continuity Keeper v1, v2 and v3 all state that `memwal_recall` returns `distance`
(0 = identical) and build the dedup gate on it. The MCP tool returns `score`, where 1.000 = identical.

**Reproduce.**
```
memwal_recall(query="<exact text of a stored record>", namespace="<ns>", limit=5)
```

**Observed.**

| content | value returned |
|---|---|
| byte-identical text | **1.000** |
| related fact | 0.554 |
| unrelated fact | 0.175 |

**What it means.** Apply the published thresholds to those numbers:

| situation | score | rule it lands in | rule's action | correct action |
|---|---|---|---|---|
| exact duplicate | 1.000 | `≥ 0.70 → unrelated` | **writes it** | skip |
| genuinely new fact | 0.175 | `< 0.25 → duplicate` | **skips it** | write |

The gate inverts at both ends: it stores every duplicate and discards every new fact.

**Why the prior evolutions missed it.** Continuity Keeper v2 measured its thresholds and
[found them correct](https://github.com/Olalekan2345/Prompt-Evolution/blob/main/demo/calibrate.mjs)
— through the **SDK**, where `recall()` genuinely returns `distance` and the framing holds. The
prompt instructs the reader to use the **MCP** tools, where it does not. Its calibration routine
("observe both distances and place the boundaries midway") also assumes lower-is-closer, so on
MCP it yields a reversed ladder that fails silently.

**Fix.** §0 of the Calibrated Edition *detects* the scale with two probes and selects the matching
threshold column. Correct on either transport, and on any client that changes the field name.

---

## 2. `memwal_analyze` writes before dedup can run, and writes twice

**Claim.** Every version routes candidate extraction through `analyze`, then deduplicates. Analyze
stores on call, so the ladder is unreachable on the primary write path. It also double-writes.

**Reproduce.**
```
memwal_restore(namespace="<fresh-ns>", limit=100)   → total=0
memwal_analyze(text="<three-sentence passage>", namespace="<fresh-ns>")
memwal_restore(namespace="<fresh-ns>", limit=100)
```

**Observed.** Analyze reported `Extracted 3 fact(s) — succeeded=3` with three blob IDs. Restore
then reported `total=6`. Recall returned every fact in a duplicate pair at identical scores.

**Filed.** [MystenLabs/MemWal#729](https://github.com/MystenLabs/MemWal/issues/729), with the
reproduction and the distinction from #562 (`remember` idempotency, merged nine days earlier) and
#87 (extraction-cost amplification).

**Fix.** §3 removes `analyze` from the write path. The agent extracts candidates in its own
reasoning so nothing reaches storage ahead of the gate.

---

## 3. A write acknowledgment is not proof of a write

**Claim.** Continuity Keeper v2 instructs: *"Never verify a write by recalling it. Trust the
acknowledgment."* Acknowledgments lie in two different directions.

**Observed, same day.**
- A `remember` returned a job id and timed out at 90s. It never landed: `restore → total=0`, still
  0 fifteen minutes later.
- An `analyze` returned `succeeded=1 failed=5` inside a **success-shaped response**. Five facts
  failed silently; restore confirmed one blob.

**Fix.** §6 keeps per-fact receipts, an explicit UNSAVED list, and treats a job-id-then-timeout as
an unknown outcome rather than blind-retrying — a blind retry can mint a second paid blob.
