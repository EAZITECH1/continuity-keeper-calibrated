# I stopped re-explaining my project to every AI session

I wrote down a fact about Walrus storage and it was half right.

The record said storage overhead is a small multiple of the blob size rather than n-fold. True, for large blobs. What it left out: every blob carries a fixed metadata cost of roughly 64 MB whatever its size, and below about 10 MB that fixed cost swallows everything else. My readers were going to be storing small things. JSON documents, memory records. The half I'd written down was the half that didn't apply to them.

I'm a Pharm.D student at LASUCOM in Lagos and I write Web3 explainers on the side. Same protocols, different clients. Corrections never survived the session. I'd fix something on Sunday and Wednesday's draft had no idea Sunday happened.

For this piece, a developer explainer on Walrus, I ran an evolved version of a prompt called Continuity Keeper with its memory stored on Walrus itself. The idea is narrow. Before the agent writes anything it recalls what's already established, and before it finalises it checks the draft against that. Facts live in an encrypted namespace I own rather than a chat history I lose.

So, the storage fact.

I'd filed it early in the evening. Hours later, working through reference material, the agent stopped and raised it as a conflict against my own canon. It quoted the stored record back at me, explained which range it was wrong in, and asked whether to revise. I said supersede. It wrote a new record covering both regimes and kept the old one on Walrus as history instead of deleting it. Both surface together on recall now, so nothing picks up the retired version without seeing what replaced it sitting right beside it.

Then I closed Claude Code, opened Codex, typed `boot`. Different vendor, different model. No shared context with the session I'd just ended. It loaded the namespace, inherited the calibration, listed the two verification items still unresolved, and when I asked it to write that Walrus fully replicates blobs across all nodes, it refused and said why.

It also failed once.

The corrected record contains a literal instruction: never state the cost story without the fixed-overhead half. While drafting the 1,290-word piece, working from that exact record, the agent wrote an unqualified cost claim anyway. One question surfaced it and the fix was clean. But an instruction sitting inside a memory record doesn't enforce itself; it's context, and context loses to whatever reads more fluently. Rules belong in the prompt. Records hold facts.

Seventeen blobs across three namespaces now. Thirteen live facts, two chronicle entries, one calibration.

I don't write faster. I've stopped re-teaching.
