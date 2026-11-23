# Gjallar HTTP API

Gjallar answers one question per address: is this address on the operator's screening
list? Clients send the chain and address of each inbound sender; Gjallar answers within a
fixed deadline and never fails a batch because of one bad item. Unknown response fields
must be ignored by clients.

## Listeners and auth

- **Node listener** (`server.bind`): `POST/GET /v1/check`, `GET /v1/list/{chain}`,
  `GET /healthz` (status only).
- **Admin listener** (`server.admin_bind`, loopback by default): `GET /v1/version`,
  `GET /v1/export/{chain}`, `GET /healthz/detail`, `GET /metrics`.
- **Auth modes** (`server.auth.modes`, any combination): `token`
  (`Authorization: Bearer <token>`, stored as sha256 digests) and `cidr` (source-IP
  allowlist; the socket peer unless `trusted_proxies` is set, in which case
  `X-Forwarded-For` is walked right to left through the trusted proxies only). Admin routes
  need an `admin` principal. Unauthenticated requests get `401`.
- A non-loopback bind requires TLS (`server.tls`). `SIGHUP` reloads tokens, the allow and
  stop lists, and revocations without a restart.

## POST /v1/check

Request:

```json
{"items": [
  {"chain": "ETH", "address": "0x…", "block_time": 1700000000, "height": 19000000, "tx_id": "0x…"},
  {"chain": "BTC", "address": "bc1q…"}
]}
```

Only `chain` and `address` are required. `block_time` (unix seconds), `height`, `tx_id`,
`amount` and `asset` are optional and may be `null`. The body may be up to 4 MiB; the item
count is never rejected.

Response (always within `request_deadline_ms`, default 1500 ms):

```json
{"list_version": "<own-set version>",
 "results": [
   {"chain": "ETH", "address": "0x…", "listed": true, "tier": "own",
    "sources": ["<label>"], "first_listed_utc": "2023-11-14T22:13:20Z"},
   {"chain": "BTC", "address": "bc1q…", "listed": false, "tier": "unverified",
    "reason": "vendor_down", "retry_after_ms": 3000}
 ]}
```

- One result per item, in request order. `address` is echoed exactly as sent; match
  results by `chain` and `address`.
- `tier`:
  - `whitelist`: a THORChain vault, router or module address, or an operator allow-list
    entry. Always `listed: false`, never sent to a vendor.
  - `own`: on the operator's list. `listed` is dated by `block_time` when both it and the
    entry's first-listed time are known.
  - `vendor`: decided by the configured vendor (`listed` = score at or above the
    threshold).
  - `unverified`: no decision in time; `listed: false`, with `reason` and
    `retry_after_ms`.
  - `invalid`: malformed address or unknown chain; `listed: false`, with `reason`.
- `reason` values: `vendor_down`, `vendor_slow`, `deadline`, `over_budget`,
  `vendor_unsupported_chain`, `whitelist_stale`, `historical`, `pending_cap`,
  `not_configured`, `malformed`, `unknown_chain`, `below_floor`, `vendor_auth`,
  `quarantined`.
- While the vault whitelist isn't reconciled to the chain tip, no result is
  `listed: true` (`unverified`, reason `whitelist_stale`).
- `sources` carries operator-set labels; sources not marked exportable appear as the single
  label `private`. `sources` and `first_listed_utc` (RFC 3339) are present only when known;
  `pinned: true` marks a result served from a per-inbound pin.
- Only the first 1000 distinct vendor misses per request go to the vendor; the rest are
  `unverified`.

## GET /v1/check?chain=&address=[&block_time=]

Single-item form of the above; same result object plus `list_version`.

## GET /v1/list/{chain}

```json
{"list_version": "…", "list_fingerprint": "…", "addresses": ["…"]}
```

Addresses from exportable sources only; whitelisted addresses are never included. A strong
`ETag` is returned; `If-None-Match` with the current tag gives `304`. `list_fingerprint` is
the sha256 over the sorted rows `key_kind | chain | canonical key bytes` of the exportable
set and is shared, byte for byte, with the bifrost adapter's list source. Clients learn
removals by diffing two responses.

## Admin

- `GET /v1/version`: current version, `list_fingerprint`, `config_fingerprint` (sha256 of
  the config file the process started with), `config_reloaded_fingerprint` (the file the
  last SIGHUP reload applied, or null), own-list size and vault-whitelist freshness.
- `GET /v1/export/{chain}`: exportable own-list entries with provenance (`address`,
  `sources`, `first_listed_utc`, `hops`).
- `GET /healthz/detail`: status and reasons. Degraded reasons: stale or quarantined vault
  whitelist, whitelist additions the confirm source hasn't matched, a source whose refresh
  is failing, a source version held as suspect, a truncated derivation, a failing set
  build, a vendor breaker not closed, vendor auth errors, vendor budget at 80 %, and an
  `unverified_ratio` (vendor-side unverified answers over tier-2 answers in the window)
  above `[health] unverified_ratio_max`. With `[health] strict_health = true`, both health
  endpoints answer 503 while tier 2 is down (a breaker open or vendor auth errors).
- `GET /metrics`: counters.

## Matching

Addresses are compared in canonical per-chain form: EVM 20-byte accounts across every EVM
chain, bech32/bech32m lowercase, BCH cashaddr with or without its prefix, base58 after
checksum, UTXO scripts by script bytes. A form without a decoder is compared as a
normalized string.
