# Gjallar

A backend service that answers "should this sender address be screened?" for a batch of
addresses, in fixed tiers:

0. **Protocol-vault whitelist**: addresses the protocol itself controls are never listed and
   never sent anywhere.
1. **Own list**: a versioned set built from configured sources (sanctions lists, CSV/JSON/web
   sources, fixed lists) plus bounded N-hop derivation along on-chain fund flows.
2. **Vendor**: a cached, coalesced lookup against an external screening service; on vendor
   trouble the answer is `unverified` (fail-open).

## Build and test

    scripts/ci.sh          # fmt, clippy, tests, cargo-deny / cargo-audit when installed
    scripts/release.sh     # release build with build paths remapped

See `API.md` for the HTTP API and `config.example.toml` for every setting.

## Known limitations

- **Derivation heights are not pinned.** The set builder reads each chain at its latest
  height (`as_of` is recorded as unbounded). Two builds from the same sources therefore
  match only when the chain data they read is identical. The adapters already filter by
  `as_of`; pinning needs a per-chain tip query in each adapter, read once at build start
  and stored in the version's inputs.
- **No vendor-control client.** `serve` runs the derivation with no external control
  policy client (`NoControl`); per-source exclude/tag policy still applies.
- **A derive error fails the whole build.** With derivation on, a chain-data error for one
  source keeps the last published version serving (it is restored from `main.db` after a
  restart) but blocks new source rows from publishing until the indexer recovers.
- **Whitelist freshness trusts the primary's tip.** Freshness compares the cursor with the
  tip of the same provider, so a provider that answers but lags isn't detected. A cursor
  height the provider has pruned keeps the guard not fresh until the cursor is reset by
  hand; there is no reseed command yet.
- **The first OFAC version publishes unstaged.** Later versions are gated (shrink against
  the week's largest version, `max_added` default 250, both counted in addresses, not chain
  rows); the very first one isn't, so a fresh install trusts its first sanctions fetch.
- **Firewall the node port.** Per-peer caps (`max_connections_per_peer`,
  `max_tls_handshakes_per_peer`, 8 handshakes) under a larger handshake budget
  (`max_tls_handshakes`, 256) keep a few peers from starving the validator's client, and a
  stalled handshake never takes a connection slot. A flood from many addresses can still fill
  `max_connections` or the handshake budget, and an unreachable Gjallar means bifrost fails
  open: allow only the validators that query it. Peers are told apart by source IP (IPv6 per
  /64), so **don't put a load balancer, NAT or proxy in front**: every client behind it
  counts as one peer, and a single client at the per-peer cap locks the others out.
- **HTTP/2 is off by default** (`server.http2 = false`: ALPN offers only http/1.1 and a
  prior-knowledge HTTP/2 preface is refused). An HTTP/2 client can keep a connection open
  with PING frames alone, which the idle timer counts as traffic; with `http2 = true` such
  a connection is held until it closes or `max_connections_per_peer` stops more of them.
- **Open-files limit.** At start the soft open-files limit is raised toward the hard limit
  (up to 65536). If the hard limit is also low, `max_connections` is lowered to fit and a
  warning is logged; raise `LimitNOFILE` in the service unit.
- **OFAC rows.** Addresses are taken from the `Digital Currency Address - <TICKER>` remarks
  (SDN.CSV plus its overflow file, fetched as a consistent pair: the main file is fetched
  again after the overflow, and a publish in between triggers one retry, then a skipped
  refresh). A trailing full stop is dropped; XRP is mapped; USDT/USDC go by address shape
  (0x = EVM, T = TRON, 1/3/bc1 = Bitcoin, Omni). An address that doesn't fit its ticker's
  chain (OFAC's own mis-tags, e.g. a TRON address tagged XBT) is quarantined and counted in
  the skip report, not guessed onto another chain.
- **Shrink baseline after an upgrade.** A baseline saved by a build before the address-count
  gate (row counts) is re-based on the published set, in addresses, when `main.db` opens.
- **Whitelist additions need both sources.** The vault history pull whitelists only the
  pubkeys and routers the primary and the confirm source both list (at most 512 per pull);
  the rest are quarantined and shown in health. The 60 s backstop may add at most 32 new
  vaults and 2 new routers per chain per pass, and 64 vaults and 4 routers per chain per
  24 h; those running counts live in memory and restart from zero after a restart. An entry
  counts as a new vault only if it brings an address the whitelist doesn't hold (SOL is
  listed under the vault's EdDSA key in /inbound_addresses, the same vault). A primary's
  addition is confirmed by the confirm source's vault view or its /inbound_addresses view,
  so a router upgrade that its vault view hasn't caught up with doesn't page.
- **Vendor generation counters aren't pruned.** One small counter per screened key stays in
  memory for the process lifetime; it matters only with a vendor configured.
- **WAL files.** `main.db` runs in WAL mode: back up `main.db-wal` and `main.db-shm` with it.
