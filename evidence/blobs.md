# Mainnet blob evidence

Every record below was written by the Calibrated Edition during real work on `walrus-notes`, a
technical explainer on Walrus. Not a seeding script — these came out of drafting an actual article.

**Live counts, verified with `memwal_restore` at submission time:**

| namespace | blobs |
|---|---|
| `walrus-notes::canon` | 13 |
| `walrus-notes::chronicle` | 2 |
| `walrus-notes::meta` | 2 |
| **total** | **17** |

## `walrus-notes::canon`

Project-level:

| record | blob |
|---|---|
| `[decision\|settled]` premise — technical explainer on Walrus | [`l3w97-Xt…`](https://walruscan.com/mainnet/blob/l3w97-XtvAllrKw7TsnZNFdEzHjOoF7XMMvhOSHMo40) |
| `[decision\|settled]` audience — mechanism + comparative rationale | [`iIQnHWjD…`](https://walruscan.com/mainnet/blob/iIQnHWjDADnEgbHVGcVEjPTdVWDjUJkD6vBj5HAOxEI) |
| `[preference\|settled]` never use vague analysis | [`STDwWQf3…`](https://walruscan.com/mainnet/blob/STDwWQf3SYQhHO_MnsXnNKGNXTxEthpKmtd7M_3z3yw) |

Technical canon:

| record | blob |
|---|---|
| ✓ **current** — blob storage: erasure coding + two cost regimes | [`xTXpDmKo…`](https://walruscan.com/mainnet/blob/xTXpDmKocDevqz_rZDFll9Q8f1J7f6rHoQVoTfIpB9o) |
| ↻ **superseded** — blob storage: cost as small multiple only | [`XVqMjmRD…`](https://walruscan.com/mainnet/blob/XVqMjmRDiVgkfzjOqIOPUeg84oFkUNV66i2i8tqbkU8) |
| blob visibility — public by default, no access control | [`1KOFCAQW…`](https://walruscan.com/mainnet/blob/1KOFCAQWi4eMrsUN2pKIgSVACflnmS3av_5jOCpQf90) |
| memory encryption (Seal) — MemWal layer, not Walrus | [`yrPBLzDG…`](https://walruscan.com/mainnet/blob/yrPBLzDGKB471XrIM1MkgMu1D9KxViTHkkfFwn_XolM) |
| blob ID vs Sui object ID | [`czhzt1Mj…`](https://walruscan.com/mainnet/blob/czhzt1MjBPC-M15Jed7StkNOXxqE0MwlrFMjlHYuwGs) |
| permanent-blob — deletable is the default | [`9rgIHBn7…`](https://walruscan.com/mainnet/blob/9rgIHBn7x77TeiB7yywZOOZx1b18G0kKT45Tfw8gb8s) |
| storage-is-time-limited — 14-day epochs, 53 max | [`KcOtehDt…`](https://walruscan.com/mainnet/blob/KcOtehDt8zo3GVs7j3jtzJwIBJNul-X02gHv7cGl1KU) |
| dual-token — WAL for storage, SUI for gas | [`Kkt1bfq8…`](https://walruscan.com/mainnet/blob/Kkt1bfq8nv_N2jmf-aQPguPHh0BaHeXl06p_gpn9suk) |

**The superseded pair is the point.** Both blobs are live on Walrus. The retired one was never
deleted, because these tools cannot delete. It stops being canon because every recall that
surfaces it also surfaces its replacement.

## `walrus-notes::meta`

| record | blob |
|---|---|
| calibration — SIMILARITY scale, `N_dup 0.815` / `N_unrel 0.145` | [`GaFCq9DL…`](https://walruscan.com/mainnet/blob/GaFCq9DLZVruU7rBHSx1F6Cd_vE4grhbZZBST-6sFaE) |

A second session in a different client inherited this calibration instead of re-probing.
That is the record that made it possible.
