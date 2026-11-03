# bifrost-filter

Opt-in, bifrost-only inbound **address filter** for THORChain and MAYAChain nodes.
Off by default. When enabled, inbound observations whose sender is on the list are not
observed by the node; outbounds, vault migrations and keysign are unaffected.

Each release ships the two patched binaries, the official Dockerfile with only those
binaries swapped, the patch, and SHA256SUMS. Images are on Docker Hub.

## THORChain

| version | docker image |
|---|---|
| 3.21.1 | `docker.io/an0n0p3rat0r/thornode:mainnet-3.21.1-filter@sha256:1f6f8d05f5e97d22fd04870e8d45fca4a3c0a39147b4e3aeb01534271a91b86f` |

See `thorchain/` for the patch, setup and address list.

## MAYAChain

| version | docker image |
|---|---|

See `maya/` for the patch, setup and address list.

