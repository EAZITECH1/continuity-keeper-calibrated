### 20:00 — guard fires against my own canon
```
⚠️ **Continuity conflict — and this one needs your call.**

> **Canon:** «Walrus blob storage — storage overhead is a small multiple of the blob size rather than n-fold» (blob `XVqMjm…`, filed this session)
> **Reference:** each blob carries ~64 MB of metadata overhead *regardless of size*, and for blobs under ~10 MB that fixed cost dominates. Quilts exist specifically to amortize it across many small blobs.

Both are true in their own range, but the canon record as written is misleading exactly where your readers will land — a memory record, a JSON doc, a small image. "Small multiple of blob size" describes the large-blob regime only.

Retcon it? I'd supersede with a record that states both regimes: overhead is a small multiple of content size for large blobs, plus a fixed ~64 MB per-blob metadata cost that dominates below ~10 MB. That keeps the erasure-coding claim intact and stops the cost story from being wrong for small payloads. Say the word and I'll write the supersession with the old wording repeated so both surface together.
```

### 20:30 — supersession receipt
```




`↻ superseded: Walrus blob storage — cost claim now carries both regimes (small multiple of size + fixed ~64 MB per blob) → blob xTXpDm…`

I spent one extra recall verifying the supersession actually behaves, because this is the one mechanism where a silent failure poisons every future session. It works: the new record lands at **0.790** and the retired one at **0.756**, adjacent in the same result set. Newest `as of` wins, and the old wording is right there next to it — so a future session can't pick up the retired cost claim without also seeing what replaced it. That's the whole reason §5 asks for the old phrasing to be repeated verbatim.

```
