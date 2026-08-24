# Continuity Keeper — Calibrated Edition

An AI co-writer that keeps a persistent, wallet-owned **canon** on Walrus Mainnet: it recalls
established facts before it drafts, refuses to contradict them, and records corrections so they
survive the session, the app, and the vendor.

Evolution of [Continuity Keeper](https://github.com/yukitran03/continuity-keeper) by yukitran03.
**→ [The prompt](prompt/continuity-keeper-calibrated.md)** (copy-paste, no CLI, no repo required)

---

## Verify this in five minutes

Nothing here asks you to take my word for it.

1. **Open any blob** in [evidence/blobs.md](evidence/blobs.md). It resolves on Walruscan or it doesn't.
2. **Find the superseded pair** in that file: [`xTXpDm…`](https://walruscan.com/mainnet/blob/xTXpDmKocDevqz_rZDFll9Q8f1J7f6rHoQVoTfIpB9o) (current) and [`XVqMjm…`](https://walruscan.com/mainnet/blob/XVqMjmRDiVgkfzjOqIOPUeg84oFkUNV66i2i8tqbkU8) (retired). Both live. One is canon.
3. **Reproduce finding 1** with one call — the exact command is in [evidence/findings.md](evidence/findings.md). If identical text returns 1.000 on your client, the published ladder writes duplicates.
4. **Read what failed** in [evidence/failure.md](evidence/failure.md).

| | |
|---|---|
| Blobs on Mainnet | **17** — 13 canon, 2 chronicle, 2 meta |
| Agent ID | `<FILL: MEMWAL_AGENT_ID>` |
| MemWalAccount | `<FILL: suiscan link>` |
| Namespaces | `walrus-notes::canon` · `::chronicle` · `::meta` |
| Issue filed during the build | [MemWal#729](https://github.com/MystenLabs/MemWal/issues/729) |
| Demo video | `<FILL>` |
| Article | `<FILL: Medium link>` |

---

## What changed, and why

Three defects survive in every published version of this prompt, including both prior evolutions.
All three were measured against the live relayer. Full reproductions in
**[evidence/findings.md](evidence/findings.md)**.

**1 — The dedup ladder is inverted on the MCP path.** Every version states `memwal_recall` returns
`distance` (0 = identical). It returns `score`: identical text measures **1.000**, unrelated
**0.175**. Read the published thresholds against those numbers and `< 0.25 → SKIP` discards new
facts while `≥ 0.70 → WRITE` stores exact duplicates. §0 detects the scale instead of assuming it.

**2 — `memwal_analyze` writes before dedup can run, and writes twice.** One call reporting *"3
facts, 3 blob_ids"* left **6 blobs** on chain. The dedup gate every evolution refined is
unreachable on the primary write path. §3 removes it.

**3 — A write acknowledgment is not proof of a write.** A `remember` returned a job id, timed out,
and never landed. An `analyze` returned `succeeded=1 failed=5` inside a success-shaped response.
§6 tracks unconfirmed writes without blind-retrying them.

Also added: restore-on-cold-recall (an empty index is not an empty namespace), a session boot that
loads state before the first reply, standing preferences captured on the second correction, and a
`memory` audit that lists the session's writes with Walruscan links.

---

## Evidence

| | |
|---|---|
| [evidence/blobs.md](evidence/blobs.md) | Every blob, with links. Live counts. The superseded pair. |
| [evidence/findings.md](evidence/findings.md) | Three findings, each with the exact call to disprove it |
| [evidence/failure.md](evidence/failure.md) | **What did not work.** The prompt's own reproducible failure |
| [evidence/session-excerpts.md](evidence/session-excerpts.md) | Verbatim: the guard firing against my own canon, the supersession receipt, the cross-client scoreboard |
| [evidence/article-drafted-by-prompt.md](evidence/article-drafted-by-prompt.md) | The 1,290-word explainer the prompt produced |
| [evidence/article-published.md](evidence/article-published.md) | My write-up of using it |
| [feedback/memwal-issue-729.md](feedback/memwal-issue-729.md) | The issue filed during the build |
| [prior-work.md](prior-work.md) | Credit, and what this edition adopts rather than reinvents |

---

## What it looked like in use

I write Web3 explainers. For this one I filed a fact early: *storage overhead is a small multiple
of the blob size rather than n-fold.* True for large blobs. Every blob also carries a fixed ~64 MB
metadata cost that dominates below ~10 MB, and my readers were going to be storing small things.

Hours later, reading reference material, the agent raised it as a conflict **against my own
canon** — quoted the record back, named the range it was wrong in, and asked whether to revise. It
superseded the record and kept the old one as history.

Then I closed Claude Code, opened Codex, and typed `boot`. Different vendor, different model, no
shared context. It loaded the namespace, inherited the calibration rather than re-probing, listed
the two verification items still open, and refused to write that Walrus fully replicates blobs.

It also failed once, drafting an unqualified cost claim from the very record that forbade it. That
is in [evidence/failure.md](evidence/failure.md), because a prompt you can only see succeed is one
you cannot calibrate.
