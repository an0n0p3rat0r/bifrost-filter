# THORChain bifrost filter setup (3.21.4)

## 1) Go to the node folder

```
cd ~/<node>/node-launcher
```

## 2) Edit the image

```
nano thornode-stack/mainnet.yaml
```

edit new 3.21.4 bifrost image:

```yaml
    - height: 28366100
      image: registry.gitlab.com/thorchain/thornode:mainnet-3.21.4@sha256:3be00fc4b3fc7dcaec18cfe0f5524bba476026302b8a356a34825ff21b9fbfb2
      containers:
        #bifrost: registry.gitlab.com/thorchain/thornode:mainnet-3.21.4@sha256:3be00fc4b3fc7dcaec18cfe0f5524bba476026302b8a356a34825ff21b9fbfb2
        bifrost: docker.io/an0n0p3rat0r/thornode:mainnet-3.21.4-filter@sha256:cdb0ae8b97403a105990120ecf1c9e1fbd2c770445e200bdf17c2cada8306f7f
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
