# Maya bifrost filter setup

## 1) Go to the node folder

```
cd ~/<node>/node-launcher
```

## 2) Edit line 84 of the bifrost chart

```
sed -i.bak '/coalesce .Values.global.hash .Values.image.hash/s/.*/{{- .Values.image.repository -}}:{{ if .Values.image.hash }}{{ .Values.image.tag }}{{ else }}{{ include "bifrost.tag" . }}{{ end }}@sha256:{{ coalesce .Values.image.hash .Values.global.hash }}/' bifrost/templates/_helpers.tpl
```

## 3) Add/edit the following values in `mayanode-stack/chaosnet.yaml`

```yaml
bifrost:
  image:
    repository: docker.io/an0n0p3rat0r/mayanode
    tag: mainnet-1.135.2-filter
    hash: b452b2ee105163af93a56fe83b509a0af996fd68815ff74d4fe110b9957564e5
  env:
    # Filters
    BIFROST_ADDRESS_FILTER_ENABLED: "true"
    BIFROST_ADDRESS_FILTER_ADDRESSES: "WALLET_ADDRESS_LIST_HERE"
```

## 4) Check

```
kubectl -n <node> logs deploy/bifrost -c bifrost | grep -i address_filter
```

After 2-3 min, expect `"loaded":627`.
