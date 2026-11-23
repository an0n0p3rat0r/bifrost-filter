# Screening (optional)

Dynamic, sourced screening on top of the static address filter. Off by default; per-node policy
(no consensus or economic change). Two pieces:

- **bifrost filter+screening build** — `docker.io/an0n0p3rat0r/thornode:mainnet-3.21.5-filter-screening`
  (`sha256:f7352575a716910af75c1360614dd62e7c722718edce402c62479c360b9cc61b`). Same as the `-filter`
  build plus a client that queries a local Gjallar service; includes the outbound-hold fix.
- **Gjallar** — `docker.io/an0n0p3rat0r/gjallar:0.1.0-ca04ded`
  (`sha256:3ab4d05f0035e96a70837cf9d44b338f56dae535e3cc1de7e8db9372a4b93236`). A local screening
  service that answers "is this sender listed?" in tiers: protocol-vault whitelist (never listed,
  never sent anywhere) → own list (OFAC + configured sources, optional N-hop) → optional external
  vendor (default off). Pulls lists in; sends nothing out unless you configure a vendor.

## Files
- `setup.md` — how to wire it up.
- `filtering-3.21.5-screening.patch` — the bifrost source patch (provenance).
- `gjallar-chart.patch` — node-launcher chart patch that adds the optional Gjallar pod.
- `gjallar/` — Gjallar docs and example config (`README.md`, `API.md`, `config.example.toml`).
