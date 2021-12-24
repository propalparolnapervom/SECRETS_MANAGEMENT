
## Start/Stop

### "Dev" mode

[Docs: 'Dev' Server](https://www.vaultproject.io/docs/concepts/dev-server)
[Docs: Starting 'Dev' Server](https://learn.hashicorp.com/tutorials/vault/getting-started-dev-server?in=vault/getting-started)

This dev-mode server requires no further setup, and your local vault CLI will be authenticated to talk to it.

Every feature of Vault is available in "dev" mode. 

In this mode, Vault runs entirely in-memory
and starts unsealed with a single unseal key. The root token is already
authenticated to the CLI, so you can immediately begin using Vault.


The `-dev` flag just short-circuits a lot of setup to insecure defaults.
> **NOTE**: Never, ever, ever run a "dev" mode server in production. It is insecure and will lose data on every restart (since it stores data in-memory). It is only made for development or experimentation.
```
vault server -dev

# Once Dev-server started ...

# ... set Vault server address
export VAULT_ADDR='http://127.0.0.1:8200'

# ... set var for Root Token
export VAULT_TOKEN="s.XmpNPoi9sRhYtdKHaQhkHP6x"

# Save the unseal key somewhere. Don't worry about how to save this securely. For now, just save it anywhere. 
```

## Show

Status
```
vault status

      Key             Value
      ---             -----
      Seal Type       shamir
      Initialized     true
      Sealed          false
      Total Shares    1
      Threshold       1
      Version         1.9.2
      Storage Type    inmem
      Cluster Name    vault-cluster-72b52d71
      Cluster ID      eaa1ff8e-c607-fca6-df97-7b2d4a05725e
      HA Enabled      false
```


## Seal/Unseal



### Unseal

[Docs](https://www.vaultproject.io/docs/concepts/seal#unsealing)
This process is stateful: each key can be entered via multiple mechanisms on multiple computers and it will work.


Once a Vault node is unsealed, it remains unsealed until one of these things happens:

- It is resealed via the API (see below).
- The server is restarted.
- Vault's storage layer encounters an unrecoverable error.


Unseal
```
vault operator unseal
```

### Seal
[Docs](https://www.vaultproject.io/docs/concepts/seal#sealing)

> NOTE: Sealing only requires a single operator with root privileges.

Seal
```
vault operator seal
```


### Auto Unseal
[Docs](https://www.vaultproject.io/docs/concepts/seal#auto-unseal)

This feature delegates the responsibility of securing the unseal key from users to a trusted device or service. At startup Vault will connect to the device or service implementing the seal and ask it to decrypt the master key Vault read from storage.




## Secret Engines


### Key/Value

#### Enable engine

```
vault secrets enable kv-v2

      # OR

vault secrets enable -version=2 kv
```


#### Work with secrets: CLI

[Docs](https://www.vaultproject.io/docs/secrets/kv/kv-v2#writing-reading-arbitrary-data)
[Docs: Convenient list of commands](https://learn.hashicorp.com/tutorials/vault/getting-started-first-secret?in=vault/getting-started#key-value-secrets-engine)

List (all secrets under the `secret`, which is root path for `K/V` engine)
```
# vault kv list <some/path/starting/from/'secret'>
vault kv list secret
```

Write
```
vault kv put secret/xbs_first_secret_via_cli xbs_key_name=xbs_key_value
```

Read
```
vault kv get secret/xbs_first_secret
```

#### Work with secrets: API

[Docs](https://www.vaultproject.io/api/secret/kv/kv-v2)

The KV secrets engine has a full HTTP API.




