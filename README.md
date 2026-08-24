# Continuity Keeper — Calibrated Edition

An AI co-writer that keeps a persistent, wallet-owned **canon** on Walrus Mainnet: it recalls
established facts before it drafts, refuses to contradict them, and records corrections so they
survive the session, the app, and the vendor.

**→ [The prompt](prompt/continuity-keeper-calibrated.md)** — copy-paste, no CLI, no companion scripts.

---

## For judges — 5-minute verify

| | |
| --- | --- |
| 📋 Prompt | [`prompt/continuity-keeper-calibrated.md`](prompt/continuity-keeper-calibrated.md) — copy-paste, name a project slug, done |
| 🎬 Demo | [I Corrected One Fact in Claude Code. Codex Remembered It.](https://www.youtube.com/watch?v=kas0uM6i0yc) |
| ⛓️ On-chain account | [`MemWalAccount` on Suiscan](https://suiscan.xyz/mainnet/object/0x5d253ff944ef3e1fa3b5f7bed097da86675a05f138d123a22a9d1ecd453c58f1) — active, delegate key registered |
| 🧠 Blobs | **17** on Mainnet, written while drafting a real article — not a seeding script. [Every blob ID ↓](evidence/blobs.md) |
| 🐛 Issues filed during the build | [MemWal#729](https://github.com/MystenLabs/MemWal/issues/729) (**closed as completed**) and [#762](https://github.com/MystenLabs/MemWal/issues/762), both with reproductions — [all feedback ↓](feedback/) |

Nothing here asks you to take my word for it:

1. **Open any blob** in [evidence/blobs.md](evidence/blobs.md). It resolves on Walruscan or it doesn't.
2. **Find the superseded pair**: [`xTXpDm…`](https://walruscan.com/mainnet/blob/xTXpDmKocDevqz_rZDFll9Q8f1J7f6rHoQVoTfIpB9o) is current, [`XVqMjm…`](https://walruscan.com/mainnet/blob/XVqMjmRDiVgkfzjOqIOPUeg84oFkUNV66i2i8tqbkU8) is retired. Both live on Walrus. One is canon.
3. **Reproduce finding 1** with a single call — exact command in [evidence/findings.md](evidence/findings.md).

| | |
| --- | --- |
| Agent ID | `93bca773f02940998084a169d40663f4a986aa0d59f4528efcef53747816f378` |
| MemWalAccount | [`0x5d253ff9…`](https://suiscan.xyz/mainnet/object/0x5d253ff944ef3e1fa3b5f7bed097da86675a05f138d123a22a9d1ecd453c58f1) |
| Sessions wallet | `0x9efad55f790d1f38647c565aa564cb381e145d3e7b635bb0538fcc0429f55a34` |
| Namespaces | `walrus-notes::canon` · `::chronicle` · `::meta` |
| MemWal package | `0xe7c16fbea0560e7057e2bf7422feaa4fb313749fc69c9e9092fac7a33b81d7f5` |
| Submission | [DeepSurge](https://www.deepsurge.xyz/projects/fc7587bf-8c8f-4656-bcb1-25bf2b34d3f9) |
| Article | [Every new AI session started from zero until I fixed it with Walrus Memory](https://medium.com/@ajayiisrael523/every-new-ai-session-started-from-zero-until-i-fixed-it-with-walrus-memory-784e68df004b) |

---

## What it does

**Who:** anyone writing long-running work with an AI — an explainer series, documentation, a
client's messaging, a novel. Anywhere contradicting your own earlier work is the failure that costs
you.

**Pain:** every session starts from zero. You re-explain the subject, and every draft is a fresh
chance to repeat a mistake you already fixed. I write Web3 explainers about the same protocols
month after month; a correction I made on Sunday was invisible to Wednesday's draft.

**Solution:** paste one system prompt. The agent recalls established facts before it writes a word,
stops and asks when a draft conflicts with them, and files corrections as superseding records so
the old version stops being the answer without being deleted. Because the canon lives on Walrus
rather than in a chat history, a different app on a different vendor's model reads the same facts.

---

## Quick setup

1. Install the Walrus Memory MCP server:

```json
{
  "mcp": {
    "memwal": {
      "type": "local",
      "command": ["npx", "-y", "@mysten-incubation/memwal-mcp"],
      "enabled": true
    }
  }
}
```

Claude Code plugin route: https://docs.wal.app/walrus-memory/mcp/claude-code

2. Copy the full prompt from [`prompt/continuity-keeper-calibrated.md`](prompt/continuity-keeper-calibrated.md) into your `CLAUDE.md`, `AGENTS.md`, or agent system prompt.

3. Run `memwal_login`, approve in your browser wallet.

4. Start a session with `boot`. First run asks three questions and no more: what the work is, who it's for, and what it should never do. Everything else it infers from working with you.

5. Give it a project slug once — lowercase, hyphens. Canon lives in `{slug}::canon`, `::chronicle`, `::meta`.

6. Write. Correct it when it's wrong. The correction is the point — that's what becomes permanent.

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

## What deliberately did not change

Restraint matters as much as the edits. These were left alone because they were already right:

| Kept | Why |
| --- | --- |
| **Recall-first, before a word is drafted** | The original's core discipline. Nothing here improves on it. |
| **The contradiction-guard stops and asks** | It never silently overrides. That choice is the whole reason the prompt is trustworthy, and weakening it to "warn and continue" would have made demos smoother and the tool worse. |
| **The note schema** — typed record, entity, current state, `as of` | Readable by a different model months later, which is the actual requirement. |
| **One fact per record** | Makes supersession possible at all. |
| **Never store prose, brainstorming, or speculation** | Canon dilutes fast. The original was right to be strict. |
| **The 0.25 / 0.55 / 0.70 threshold values** | Correct on the SDK's distance scale. The numbers were never the bug — the assumed direction was. |
| **v2's entity-name tiebreak for the ambiguous band** | A better resolution than the one I first wrote. Adopted rather than replaced. |

Full attribution in [prior-work.md](prior-work.md).

---

## Stack

| | |
| --- | --- |
| Prompt | Markdown. No code, no CLI, no repo dependency — it runs from a paste. |
| Memory | [Walrus Memory (MemWal)](https://docs.wal.app/walrus-memory) via `@mysten-incubation/memwal-mcp` |
| Storage | Walrus **Mainnet** — encrypted blobs, owner-held keys |
| Coordination | Sui — `MemWalAccount` shared object, delegate keys |
| Tools used | `memwal_recall`, `memwal_remember`, `memwal_remember_bulk`, `memwal_restore`, `memwal_health` |
| Tool deliberately **not** used | `memwal_analyze` — see finding 2 |
| Verified on | Claude Code desktop (Opus 5) and Codex desktop (5.4 Mini Light) |

The small model matters: Codex ran this on 5.4 Mini Light and held the protocol — booted, inherited
the calibration, and refused a contradiction. Long prompts usually lose discipline on small models.

---

## Evidence

| | |
| --- | --- |
| [evidence/blobs.md](evidence/blobs.md) | Every blob, with links. Live counts. The superseded pair. |
| [evidence/findings.md](evidence/findings.md) | Three findings, each with the exact call to disprove it |
| [evidence/session-excerpts.md](evidence/session-excerpts.md) | Verbatim: the guard firing against my own canon, the supersession receipt, the cross-client run |
| [evidence/article-drafted-by-prompt.md](evidence/article-drafted-by-prompt.md) | The 1,290-word explainer the prompt produced |
| [evidence/article-published.md](evidence/article-published.md) | My write-up of using it |
| [feedback/](feedback/) | Two issues filed during the build, plus four earlier ones and the rule each produced |
| [prior-work.md](prior-work.md) | Credit, and what this edition adopts rather than reinvents |

---

## What it looked like in use

I filed a fact early: *storage overhead is a small multiple of the blob size rather than n-fold.*
True for large blobs. Every blob also carries a fixed ~64 MB metadata cost that dominates below
~10 MB, and my readers were going to be storing small things.

Hours later, reading reference material, the agent raised it as a conflict **against my own
canon** — quoted the record back, named the range it was wrong in, and asked whether to revise. It
superseded the record and kept the old one as history.

Then I closed Claude Code, opened Codex, and typed `boot`. Different vendor, different model, no
shared context. It loaded the namespace, inherited the calibration rather than re-probing, listed
the two verification items still open, and refused to write that Walrus fully replicates blobs.

---

## Submission checklist

- [x] Evolution of a base prompt — [Continuity Keeper](https://github.com/yukitran03/continuity-keeper) by yukitran03
- [x] Full copy-pasteable prompt text — [`prompt/`](prompt/continuity-keeper-calibrated.md)
- [x] ≥10 blobs on Walrus Mainnet — **17** (13 canon · 2 chronicle · 2 meta)
- [x] Agent ID and blob count recorded — table above
- [x] Dedicated Sessions wallet address
- [x] Demo video — [YouTube](https://www.youtube.com/watch?v=kas0uM6i0yc)
- [x] Feedback / GitHub issues on MystenLabs/MemWal — [#729](https://github.com/MystenLabs/MemWal/issues/729) (closed as completed) and [#762](https://github.com/MystenLabs/MemWal/issues/762)
- [x] Written blog explaining use and problems solved — [`evidence/article-published.md`](evidence/article-published.md)
- [x] Article published to Medium — [link](https://medium.com/@ajayiisrael523/every-new-ai-session-started-from-zero-until-i-fixed-it-with-walrus-memory-784e68df004b)
- [x] Entered on the DeepSurge form — [project page](https://www.deepsurge.xyz/projects/fc7587bf-8c8f-4656-bcb1-25bf2b34d3f9)
- [x] Submitted via Airtable **and** WalForm
- [x] Demo posted with #Walrus on X
- [x] Walrus Discord joined
