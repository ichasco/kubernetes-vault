# Vault

## Secrets

### Install cfssl and consul

`apt install golang-cfssl consul`

### Create CA

`cfssl gencert -initca certs/config/ca-csr.json | cfssljson -bare certs/ssl/ca`

### Generate certificates Consul

```
cfssl gencert \
    -ca=certs/ssl/ca.pem \
    -ca-key=certs/ssl/ca-key.pem \
    -config=certs/config/ca-config.json \
    -profile=default \
    certs/config/consul-csr.json | cfssljson -bare certs/ssl/consul
```

`export GOSSIP_ENCRYPTION_KEY=$(consul keygen)`

### Create Consul secrets in Kubernetes

```
kubectl create secret generic consul \
  --from-literal="gossip-encryption-key=${GOSSIP_ENCRYPTION_KEY}" \
  --from-file=certs/ssl/ca.pem \
  --from-file=certs/ssl/consul.pem \
  --from-file=certs/ssl/consul-key.pem \
  -n vault
```

### Conect to UI

`kubectl -n vault port-forward consul-0 8500:8500`


### Generate Certificates Vault

```
cfssl gencert \
    -ca=certs/ssl/ca.pem \
    -ca-key=certs/ssl/ca-key.pem \
    -config=certs/config/ca-config.json \
    -profile=default \
    certs/config/vault-csr.json | cfssljson -bare certs/ssl/vault
```

### Create Vault secrets in Kubernetes

```
kubectl create secret generic vault \
    --from-file=certs/ssl/ca.pem \
    --from-file=certs/ssl/vault.pem \
    --from-file=certs/ssl/vault-key.pem \
    -n vault
```

### Unseal Vault

`export VAULT_ADDR=https://127.0.0.1:8200`
`export VAULT_CACERT="certs/ssl/ca.pem"`

`vault operator init -key-shares=1 -key-threshold=1`

`vault operator unseal`

`vault login`



https://testdriven.io/blog/running-vault-and-consul-on-kubernetes/

