thornode 3.21.4 filtering build (opt-in, bifrost only)

Base:     thornode 3.21.4 release, commit 1d4ba603d05cec1f01c1b6a7d894423257b559b5
Patch:    filtering-3.21.4.patch (plain unified diff, applies with -p0)
Build:    linux/amd64, static (musl, static-pie), tags mainnet,muslc, garbled like
          the official release binaries. `thornode version --long` reports the
          base commit above.
Files:    thornode, bifrost, Dockerfile, filtering-3.21.4.patch, SHA256SUMS

Verify:
  sha256sum -c SHA256SUMS

Docker (official 3.21.4 image, only the two binaries replaced):
  docker build -t thornode:mainnet-3.21.4-filtering .

Rebuild from source instead (optional):
  git checkout 1d4ba603d05cec1f01c1b6a7d894423257b559b5
  git apply -p0 filtering-3.21.4.patch

Filtering (OFF by default; only bifrost changes). The config keys keep the
names used in the code:
  bifrost config:  address_filter: { enabled: true, addresses: "<comma-separated list>" }
  or env:          BIFROST_ADDRESS_FILTER_ENABLED=true
                   BIFROST_ADDRESS_FILTER_ADDRESSES="0xabc...,bc1q...,SOL:...,TRON:..."
  0x addresses apply to all EVM chains; valid BTC bc1/legacy addresses are
  inferred. Any other shape needs a CHAIN: prefix, otherwise it is skipped (the
  skip count is printed in the bifrost startup log). Inbounds whose sender is
  listed are not observed by this node. Outbounds, vault migrations and keysign
  are unaffected. A metric counts dropped inbounds.
