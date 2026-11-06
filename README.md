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
| 3.21.2 | `docker.io/an0n0p3rat0r/thornode:mainnet-3.21.2-filter@sha256:d5843ae44842ae3779ca7cfe64715dc6a48a801160bf5bb6cd67169494697444` |

See `thorchain/` for the patch, setup and address list.

## MAYAChain

| version | docker image |
|---|---|
| 1.135.2 | `docker.io/an0n0p3rat0r/mayanode:mainnet-1.135.2-filter@sha256:b452b2ee105163af93a56fe83b509a0af996fd68815ff74d4fe110b9957564e5` |
| 1.135.3 | `docker.io/an0n0p3rat0r/mayanode:mainnet-1.135.3-filter@sha256:5e2e731c48d0bf8e0036178e7ab0f4c438bf6c37ac56d4073f4bbcfb54d8adac` |
| 1.135.4 | `docker.io/an0n0p3rat0r/mayanode:mainnet-1.135.4-filter@sha256:1172ceeedd2cdd34be74cbb66ec2634161cfce32529b23779b561e1664c2560d` |

See `maya/` for the patch, setup and address list.

