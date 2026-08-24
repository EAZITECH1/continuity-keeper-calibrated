## Summary

`memwal_restore` returns `truncated=true` unconditionally, including when the namespace is empty
and when `total` is far below the requested `limit`. The MCP layer then renders the advisory
*"⚠️ More blobs remain to restore — increase limit and call again."*

The tool's own description instructs the caller to act on that flag: *"If truncated=true, increase
limit and call again."* Since the flag never clears, an agent following the documented contract
loops on a restore that is already complete, and at `limit=100` it has nowhere left to go.

## Environment

- `@mysten-incubation/memwal-mcp` **0.0.7-rc.0**, MCP tools via Claude Code
- Relayer `https://relayer.memory.walrus.xyz`, `memwal_health` → `status=ok version=0.1.0`
- Walrus **Mainnet**, 2026-08-24

## Reproduction

Real namespace, 2 blobs, asking for up to 100:

```
memwal_restore(namespace="walrus-notes::meta", limit=100)
→ total=2  restored=0  skipped=2  truncated=true
  ⚠️ More blobs remain to restore — increase limit and call again.
```

`total=2` and `limit=100`. Nothing remains, yet the flag is set.

A namespace that has never existed:

```
memwal_restore(namespace="walrus-notes-nonexistent-probe", limit=100)
→ total=0  restored=0  skipped=0  truncated=true
```

```
memwal_restore(namespace="walrus-notes-nonexistent-probe", limit=10)
→ total=0  restored=0  skipped=0  truncated=true
```

`total=0`, nothing processed, no blobs anywhere, and `truncated` is still true at both limits. The
value does not appear to be derived from the counts at all.

Three further namespaces on the same account, same call shape, same result: `13/2/2` blobs
respectively, all `truncated=true` at `limit=100`.

## Expected

`truncated` should be true only when the namespace holds more blobs than the call processed —
roughly `total > limit`, or more precisely when a further page exists. With `total=0` or
`total <= limit` it should be false, and the MCP layer should not print the "increase limit"
advisory.

## Impact

- **Agents loop.** The tool description makes the flag actionable, so a compliant agent retries a
  finished restore. At `limit=100` there is no higher limit to try, so the retry is unbounded or
  the agent stalls.
- **Cold-start recovery is where this fires.** `restore` exists for the case where recall returns
  nothing on a new machine, which is exactly when an agent is already uncertain whether memory is
  broken. A permanent "more blobs remain" turns a successful recovery into an apparent failure.
- **It also masks the genuine truncation case.** A flag that is always set carries no information,
  so a real partial restore is indistinguishable from a complete one.

## Related, not duplicates

- **#589** (merged) fixed the opposite direction: the MCP tool *dropping* the flag and reporting
  "Restore complete" on a partial restore. This is the same field failing the other way.
- **#754 / #760** cover `restored + skipped < total` with no cursor. Distinct: here `total` itself
  is 0 or fully covered by `limit`.
- **#382**, **#623** concern `total` disagreeing with what recall enumerates, not the `truncated`
  flag.
