thornode 3.21.5 filtering build (opt-in, bifrost only)

Base:     thornode 3.21.5 release, commit 59c58ecc15d5430db269b97a3d8af79ab7aa9afb
Patch:    filtering-3.21.5.patch (plain unified diff, applies with -p0)
Build:    linux/amd64, static (musl, static-pie), tags mainnet,muslc, garbled like
          the official release binaries. `thornode version --long` reports the
          base commit above.
Files:    thornode, bifrost, Dockerfile, filtering-3.21.5.patch, SHA256SUMS

Verify:
  sha256sum -c SHA256SUMS

Docker (official 3.21.5 image, only the two binaries replaced):
  docker build -t thornode:mainnet-3.21.5-filtering .

Rebuild from source instead (optional):
  git checkout 59c58ecc15d5430db269b97a3d8af79ab7aa9afb
  git apply -p0 filtering-3.21.5.patch

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
