# memwal_analyze reports N facts with N blob_ids but the namespace ends up with 2N blobs — every successfully extracted fact is stored twice

## Summary

A single `memwal_analyze` call (MCP tool, mainnet relayer) that reports **3 facts extracted,
3 succeeded, 3 blob_ids** leaves the target namespace with **6 blobs**: `memwal_restore` reports
`total=6`, and `memwal_recall` returns each fact **twice** (two hits with identical text at the
same score). The tool's own response under-reports its writes by half, so the duplication is
invisible unless you audit with `restore` — which most agents never do, especially since there is
no list operation (#626).

## Environment

- MCP tools via `memwal-mcp` (Claude Code client), production relayer, `memwal_health` → `status=ok version=0.1.0`
- Walrus **Mainnet**
- Date: 2026-08-21
- Fresh namespaces created for the reproduction (no prior contents)

## Reproduction

1. Pick a fresh namespace (mine: `s7probe3`). Confirm empty: `memwal_restore(namespace="s7probe3", limit=100)` → `total=0`.
2. Call `memwal_analyze` with a short multi-fact passage:

   > The Sunblade was forged in Ravengard. Kane carries the Sunblade after Chapter 7. The Fold exacts a memory cost from anyone who crosses it.

3. Tool response — 3 facts, all `done`, one blob_id each:

   ```
   Extracted 3 fact(s) — succeeded=3 failed=0
   1. [done] blob_id=1jS-BdqyiUA1FIIQJ7CB1_7q2IMypF2VCwcVPAjxzm8 The Sunblade was forged in Ravengard
   2. [done] blob_id=16cbYawnxiDXrg7w7j56eeOByNd6EbLFdzHb6KCA2-o Kane carries the Sunblade after Chapter 7
   3. [done] blob_id=gymbAWFe45b0h8ee51WKHLGOsR0XMhOZgY39LyG5m7E The Fold exacts a memory cost from anyone who crosses it
   ```

4. Audit the namespace:

   ```
   memwal_restore(namespace="s7probe3", limit=100)
   → total=6  restored=0  skipped=6  truncated=false
   ```

5. Recall corroborates — each fact appears twice, pairwise identical text at identical scores:

   ```
   memwal_recall(query="The Sunblade was forged in Ravengard", namespace="s7probe3", limit=5)
   1. [score=1.000] The Sunblade was forged in Ravengard
   2. [score=1.000] The Sunblade was forged in Ravengard
   3. [score=0.554] Kane carries the Sunblade after Chapter 7
   4. [score=0.554] Kane carries the Sunblade after Chapter 7
   5. [score=0.175] The Fold exacts a memory cost from anyone who crosses it
   ```

3 facts reported → 6 blobs on Walrus, 2 per fact.

## A second run that did NOT duplicate — narrows the cause

Earlier the same day, during visible relayer degradation, `memwal_analyze` on a different fresh
namespace (`s7probe2`, 6-fact passage) returned:

```
Extracted 6 fact(s) — succeeded=1 failed=5
5. [done] blob_id=-_OyaI6QPf3S7mBAMDL8tKtaBYqGKM8LEJq150ulajI ...
(other 5: [timeout])
```

Audit: `memwal_restore(namespace="s7probe2")` → `total=1`. The one successful fact was stored
**once**, and re-checking ~15 minutes later it was still `total=1`.

So the duplication is tied to the **successful completion path**, not to timeout/retry recovery:
a run where facts complete cleanly writes each twice; a run where most facts time out writes the
lone survivor once. Consistent with a pipeline that persists each fact at extraction time and
then persists the completed batch again (or an internal job that reports one write while the
worker commits two) — rather than with client-side retry.

## Impact

- **2× storage cost** — every analyze spends double the WAL/quota its response admits to.
- **Dedup logic downstream is poisoned**: any prompt that recalls before writing (recall-then-
  remember dedup gates are the standard pattern in the current Prompt Jam corpus) sees a phantom
  "existing duplicate" for facts analyze created, and skips legitimate writes.
- **Blob-count proofs are inflated** — sessions events require agents to report mainnet blob
  counts; an agent using analyze honestly reports N from tool responses while the chain holds 2N.
- **Undetectable in normal operation**: the response shows N blob_ids, there is no list op
  (#626), and only a `restore` count or an exact-text recall reveals the extra writes.

## Expected

One extracted fact → one blob, and the response's blob_id list matches what was written. If the
server intentionally writes twice (e.g. raw + normalized), the response should say so and return
both ids.

## Related — and why this is not a duplicate of them

- **#562 (merged 2026-08-12)** — made `remember` writes idempotent precisely to prevent "a second
  paid blob" from client or internal (Apalis) retries. This reproduction is from **2026-08-21,
  nine days after that merge**, on the **analyze** path: either the analyze fan-out's per-fact
  writes don't go through the guarded path, or the guard doesn't cover them. Note the shape
  differs from a retry race: in the clean run ALL 3 facts were duplicated uniformly, which looks
  more like two persistence passes (per-fact at extraction, then the completed batch) than a
  stochastic retry — while in the degraded run the job presumably died before the second pass.
- **#87 (merged April)** — "analyze amplification" is a different amplification: unbounded fact
  *extraction* cost. This report is about each successfully extracted fact being *stored* twice.
- **#626** — no list operation, which is why this is invisible in normal operation.
- **#656** — read-your-writes lag; different failure (reads missing writes, not writes doubling).
