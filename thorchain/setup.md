# THORChain bifrost filter setup

## 1) Go to the node folder

```
cd ~/<node>/node-launcher
```

## 2) Edit the image

```
nano thornode-stack/mainnet.yaml
```

edit new 3.21.1 bifrost image:

```yaml
    - height: 28366100
      image: registry.gitlab.com/thorchain/thornode:mainnet-3.21.1@sha256:73eb15224d1e797591ebeb92d057540f9c1cdcc8994d166f1acfd17a45def073
      containers:
        #bifrost: registry.gitlab.com/thorchain/thornode:mainnet-3.21.1@sha256:73eb15224d1e797591ebeb92d057540f9c1cdcc8994d166f1acfd17a45def073
        bifrost: docker.io/an0n0p3rat0r/thornode:mainnet-3.21.1-filter@sha256:1f6f8d05f5e97d22fd04870e8d45fca4a3c0a39147b4e3aeb01534271a91b86f
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
