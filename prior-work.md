# Prior work, and what this edition takes from it

This is an evolution of [**Continuity Keeper**](https://github.com/yukitran03/continuity-keeper)
by yukitran03 (Walrus Memory Prompt Jam). Its recall-first discipline, contradiction-guard and
note schema are the foundation; the design idea that canon must be *current* rather than merely
stored is entirely his.

Two other evolutions were published before this one. Both found real defects, and this edition
adopts their fixes rather than re-deriving them:

| Fix | First published by | Status here |
|---|---|---|
| The undefined 0.55–0.70 dedup band | [Continuity Keeper v2](https://github.com/Olalekan2345/Prompt-Evolution) (Olalekan2345) | Adopted, including the entity-name tiebreak — a better resolution than the one I first wrote |
| Removing the CLI dependency for supersession | Both v2 and [EvoPrompt 26 / v3](https://github.com/Olympusxvn/session7-continuity) (Olympusxvn) | Adopted |
| Consolidating per-entity namespaces | v3 | Adopted |
| Injection defence; empty ≠ failed recall | v2 | Adopted |

**What is new here** is in [findings.md](evidence/findings.md): the MCP scale inversion, the
`analyze` double-write, and the unreliability of write acknowledgments. All three survive in both
prior evolutions.

The scale inversion is the one worth stating carefully, because v2 did the work and reached the
opposite conclusion honestly. It calibrated through the SDK, where `distance` is the real field
and the published ladder is correct. The prompt tells you to run MCP tools, where the field is
`score` and the ladder inverts. Same measurement, different transport, opposite answer.
