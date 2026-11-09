thornode 3.21.3 filtering build (opt-in, bifrost only)

Base:     thornode 3.21.3 release, commit fdd91dace8608aa94ad5e0f4df0b17c15ebb66d1
Patch:    filtering-3.21.3.patch (plain unified diff, applies with -p0)
Build:    linux/amd64, static (musl, static-pie), tags mainnet,muslc, garbled like
          the official release binaries. `thornode version --long` reports the
          base commit above.
Files:    thornode, bifrost, Dockerfile, filtering-3.21.3.patch, SHA256SUMS

Verify:
  sha256sum -c SHA256SUMS

Docker (official 3.21.3 image, only the two binaries replaced):
  docker build -t thornode:mainnet-3.21.3-filtering .

Rebuild from source instead (optional):
  git checkout fdd91dace8608aa94ad5e0f4df0b17c15ebb66d1
  git apply -p0 filtering-3.21.3.patch

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
