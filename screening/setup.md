# THORChain bifrost filter + screening setup (3.21.5)

Screening adds dynamic, sourced lists (via a local Gjallar service) on top of the static filter.
Everything is opt-in and off by default.

## 1) bifrost image
In `thornode-stack/mainnet.yaml`, the 3.21.5 height entry:

```yaml
    - height: 28366100
      image: registry.gitlab.com/thorchain/thornode:mainnet-3.21.5@sha256:2bc45fffb2a1f51f2b56c4e038ad3712d767bc3cf2411d1fc230f97851080039
      containers:
        bifrost: docker.io/an0n0p3rat0r/thornode:mainnet-3.21.5-filter-screening@sha256:f7352575a716910af75c1360614dd62e7c722718edce402c62479c360b9cc61b
```

(Static-list filtering still works exactly as the `-filter` build via `BIFROST_ADDRESS_FILTER_ADDRESSES`.)

## 2) Gjallar screening service
Apply the node-launcher chart patch `screening/gjallar-chart.patch` to add the optional Gjallar pod, then set:

```yaml
gjallar:
  image:
    name: docker.io/an0n0p3rat0r/gjallar
    tag: 0.1.0-ca04ded
    hash: 65cb72553466561b5c19dfd5a759eb5c7d032eaa7d4d89b5496b31f5a62ab6f3
```

Mounts: `gjallar.toml` at `/etc/gjallar/gjallar.toml`, TLS at `/etc/gjallar/tls`, data at `/var/lib/gjallar`.
Config reference: `screening/gjallar/config.example.toml` and `screening/gjallar/API.md`.
Secrets you provide: a TLS cert/key for Gjallar (+ its CA for bifrost), bifrost→Gjallar auth token,
optional `GJALLAR_THORCHAIN_API_KEY`, optional external vendor key.

## 3) enable
Set `address_filter.enabled: true` on bifrost and point it at the local Gjallar (see the chart values).
Off by default; a node that filters simply does not sign that observation (per-node policy, no consensus change).
