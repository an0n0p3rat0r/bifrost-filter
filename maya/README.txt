mayanode 1.135.4 filtering build (opt-in, bifrost only)

Base:     mayanode 1.135.4 release, commit 90095c20459a20f28ea6375cda92269260e6d361
          (the commit the official mainnet-1.135.4 image is built from)
Patch:    filtering-1.135.4.patch (plain unified diff, applies with -p0)
Build:    linux/amd64, fully static (glibc static, netgo/osusergo), official builder-v9
          (go1.24.13), tags mainnet, -trimpath, stripped. libzec.a and
          libradix_engine_toolkit_uniffi.a rebuilt from the repo's sources with build
          paths remapped (same exported symbols as the repo's prebuilt libs).
          `mayanode version --long` reports the base commit above.
Files:    mayanode, bifrost, Dockerfile, filtering-1.135.4.patch, SHA256SUMS

Verify:
  sha256sum -c SHA256SUMS

Docker (official 1.135.4 image, only the two binaries replaced):
  docker build -t mayanode:mainnet-1.135.4-filtering .

Rebuild from source instead (optional):
  git checkout 90095c20459a20f28ea6375cda92269260e6d361
  git apply -p0 filtering-1.135.4.patch

Filtering (OFF by default; only bifrost changes). The config keys keep the
names used in the code:
  bifrost config:  address_filter: { enabled: true, addresses: "<comma-separated list>" }
  or env:          BIFROST_ADDRESS_FILTER_ENABLED=true
                   BIFROST_ADDRESS_FILTER_ADDRESSES="0xabc...,bc1q...,..."
  0x addresses apply to all EVM chains; validated unambiguous shapes (BTC, DASH,
  ZEC t-addr, bech32 chains) are inferred. Any other shape needs a CHAIN: prefix,
  otherwise it is skipped (the skip count is printed in the bifrost startup log).
  Inbounds whose sender is listed are not observed by this node. Outbounds,
  vault migrations and keysign are unaffected. A metric counts dropped inbounds.
