# Every new AI session started from zero until I fixed it with Walrus Memory

> Published on Medium: https://medium.com/@ajayiisrael523/every-new-ai-session-started-from-zero-until-i-fixed-it-with-walrus-memory-784e68df004b
> This file mirrors the published text.

---

I wrote down a fact about Walrus storage and it was half right.

The record said storage overhead is a small multiple of the blob size rather than n-fold. True, for large blobs. What it left out: every blob carries a fixed metadata cost of roughly 64 MB whatever its size, and below about 10 MB that fixed cost swallows everything else. My readers were going to be storing small things. JSON documents, memory records. The half I'd written down was the half that didn't apply to them.

I write Web3 explainers on the side for protocols, different clients. Corrections never survived the session. I'd fix something on Sunday and Wednesday's draft had no idea Sunday happened.

For this piece, a developer explainer on Walrus, I ran an evolved version of a prompt called Continuity Keeper with its memory stored on Walrus itself. The idea is narrow. Before the agent writes anything it recalls what's already established, and before it finalises it checks the draft against that. Facts live in an encrypted namespace I own rather than a chat history I lose.

So, the storage fact.

I'd filed it early in the evening. Hours later, working through reference material, the agent stopped and raised it as a conflict against my own canon. It quoted the stored record back at me, explained which range it was wrong in, and asked whether to revise. I said supersede. It wrote a new record covering both regimes and kept the old one on Walrus as history instead of deleting it. Both surface together on recall now, so nothing picks up the retired version without seeing what replaced it sitting right beside it.

Then I closed Claude Code, opened Codex, typed `boot`. Different vendor, different model. No shared context with the session I'd just ended. It loaded the namespace, inherited the calibration, listed the two verification items, and when I asked it to write that Walrus fully replicates blobs across all nodes, it refused and said why.

The corrected record contains a literal instruction: never state the cost story without the fixed-overhead half. While drafting the 1,290-word piece, working from that exact record, the agent wrote an unqualified cost claim anyway. One question surfaced it and the fix was clean. But an instruction sitting inside a memory record doesn't enforce itself; it's context, and context loses to whatever reads more fluently. Rules belong in the prompt. Records hold facts.

Thirteen live facts, two chronicle entries, one calibration.

It doesn't only make me write faster. I've stopped re-teaching my AI the same corrections.

## How to start using the prompt

Four steps.

**1. Sign in once, in a real terminal:**

```
npx -y "@mysten-incubation/memwal-mcp" login --label my-project
```

A browser opens, you approve with your wallet, and the credentials land in `~/.memwal/credentials.json`. They stay on your machine.

**2. Paste the prompt as your agent's system prompt.** One file, from the repo. `CLAUDE.md` for Claude Code, `AGENTS.md` for Codex, the system prompt field anywhere else. Nothing to configure.

**3. Give it a project name once.** It builds three namespaces from that. `myproject::canon` holds what's currently true, `myproject::chronicle` holds what happened, and `myproject::meta` holds the calibration so your next session inherits it instead of probing again.

**4. Work normally.** Tell it what you've settled and why. Correct it when it's wrong, because the correction is the part that becomes permanent. Next session, in whichever app you open, it reads all of that back before it does anything else.

Prompt and repo: https://github.com/EAZITECH1/continuity-keeper-calibrated

Demo (3 min): https://www.youtube.com/watch?v=kas0uM6i0yc
