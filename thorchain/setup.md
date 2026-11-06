# THORChain bifrost filter setup (3.21.2)

## 1) Go to the node folder

```
cd ~/<node>/node-launcher
```

## 2) Edit the image

```
nano thornode-stack/mainnet.yaml
```

edit new 3.21.2 bifrost image:

```yaml
    - height: 28366100
      image: registry.gitlab.com/thorchain/thornode:mainnet-3.21.2@sha256:6b200d2d5f8f859c07af9f923caccb17f3516ed88d9859cfcd95f073427ed9d6
      containers:
        #bifrost: registry.gitlab.com/thorchain/thornode:mainnet-3.21.2@sha256:6b200d2d5f8f859c07af9f923caccb17f3516ed88d9859cfcd95f073427ed9d6
        bifrost: docker.io/an0n0p3rat0r/thornode:mainnet-3.21.2-filter@sha256:d5843ae44842ae3779ca7cfe64715dc6a48a801160bf5bb6cd67169494697444
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
