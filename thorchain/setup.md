# THORChain bifrost filter setup (3.21.3)

## 1) Go to the node folder

```
cd ~/<node>/node-launcher
```

## 2) Edit the image

```
nano thornode-stack/mainnet.yaml
```

edit new 3.21.3 bifrost image:

```yaml
    - height: 28366100
      image: registry.gitlab.com/thorchain/thornode:mainnet-3.21.3@sha256:9f46da91327a15461f53e0d8ed2d289a0e7b562e4a552172c3e94ec99002d11c
      containers:
        #bifrost: registry.gitlab.com/thorchain/thornode:mainnet-3.21.3@sha256:9f46da91327a15461f53e0d8ed2d289a0e7b562e4a552172c3e94ec99002d11c
        bifrost: docker.io/an0n0p3rat0r/thornode:mainnet-3.21.3-filter@sha256:ac2159aab14efd4c01c56cdaa0de7003166b67476a0e737bdd1df4aa95bd17d0
```

## 3) Add/edit the following values in `thornode-stack/mainnet.yaml`

```yaml
bifrost:
  env:
    # Filters
    BIFROST_ADDRESS_FILTER_ENABLED: "true"
    BIFROST_ADDRESS_FILTER_ADDRESSES: "WALLET_ADDRESS_LIST_HERE"
```

## 4) Check

```
kubectl -n <node> logs bifrost-0 -c bifrost | grep -i address_filter
```

After 2-3 min, expect `"loaded":628`.
