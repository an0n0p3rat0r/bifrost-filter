# THORChain bifrost filter setup (3.21.5)

## 1) Go to the node folder

```
cd ~/<node>/node-launcher
```

## 2) Edit the image

```
nano thornode-stack/mainnet.yaml
```

edit new 3.21.5 bifrost image:

```yaml
    - height: 28366100
      image: registry.gitlab.com/thorchain/thornode:mainnet-3.21.5@sha256:2bc45fffb2a1f51f2b56c4e038ad3712d767bc3cf2411d1fc230f97851080039
      containers:
        #bifrost: registry.gitlab.com/thorchain/thornode:mainnet-3.21.5@sha256:2bc45fffb2a1f51f2b56c4e038ad3712d767bc3cf2411d1fc230f97851080039
        bifrost: docker.io/an0n0p3rat0r/thornode:mainnet-3.21.5-filter@sha256:ea1f87c700afa5a343357e8df4b1a67c9ffd0a8d396f74a08d5fbe22c6779b44
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
