# Feedback filed on MystenLabs/MemWal

Bugs found while building and running the Calibrated Edition, turned into tickets.

## Filed during this build

| Issue | Status | What |
| --- | --- | --- |
| [**#729**](https://github.com/MystenLabs/MemWal/issues/729) | **closed — completed** | `memwal_analyze` reports N facts with N blob IDs but leaves 2N blobs. One call reporting "3 facts, succeeded=3" left six blobs on chain. Analyze also writes at call time, so any prompt that deduplicates *after* extraction cannot work. → [full text](memwal-issue-729.md) |
| [**#762**](https://github.com/MystenLabs/MemWal/issues/762) | open | `memwal_restore` always returns `truncated=true`, even for a namespace that has never existed (`total=0`) and at `limit=100`. The tool description tells callers to raise the limit and retry whenever the flag is set, so a compliant agent loops on a finished restore. → [full text](memwal-issue-762.md) |

#729 drove a change in the prompt: §3 removes `memwal_analyze` from the write path entirely.
#762 is why §1 tells the agent to call `memwal_restore` once and then judge by what `recall`
returns, rather than trusting the flag.

## Filed earlier, still shaping this prompt

These came out of the previous Walrus Memory session. Each one is a reason a rule exists here.

| Issue | Status | What, and where it shows up |
| --- | --- | --- |
| [#414](https://github.com/MystenLabs/MemWal/issues/414) | closed — completed | `memwal_restore` returns HTTP 500 intermittently and can hang past 400s, well beyond the 30s tool timeout. Why §6 treats a timeout as an unknown outcome instead of a failure. |
| [#415](https://github.com/MystenLabs/MemWal/issues/415) | closed — completed | `memwal-mcp` blocks the MCP initialize handshake during cold start, tripping the client's 30s connection timeout. |
| [#416](https://github.com/MystenLabs/MemWal/issues/416) | closed — duplicate | `recall` and `restore` return an identical "No matching memories found" for a misspelled namespace as for an empty one. Why §1 separates **empty**, **cold** and **failed** instead of collapsing them. |
| [#580](https://github.com/MystenLabs/MemWal/issues/580) | closed — completed | In-session `memwal_login` never updates the bridge's credentials, so every later call 401s until the process is killed. |
