
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

   # OR
   
curl http://127.0.0.1:8200/v1/sys/init

      { "initialized": true }
```

Information regarding the token you eventually used to log in
```
vault token lookup

      Key                 Value
      ---                 -----
      accessor            mBTRDLasdfXamfQI3G
      creation_time       1640345928
      creation_ttl        0s
      display_name        root
      entity_id           n/a
      expire_time         <nil>
      explicit_max_ttl    0s
      id                  s.SlWawhMmO0yrdddYiM33pa
      meta                <nil>
      num_uses            0
      orphan              true
      path                auth/token/root
      policies            [root]
      ttl                 0s
      type                service
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

### General

List all secret engines enabled
```
vault secrets list

      Path          Type         Accessor              Description
      ----          ----         --------              -----------
      cubbyhole/    cubbyhole    cubbyhole_fc48f28b    per-token private secret storage
      identity/     identity     identity_5396702b     identity store
      secret/       kv           kv_d561a5f1           key/value secret storage
      sys/          system       system_5afc25c6       system endpoints used for control, policy and debugging
```

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



## Namespaces
A Vault administrator can lock the API for particular namespaces.

### Lock
[Docs](https://www.vaultproject.io/docs/concepts/namespace-api-lock#locking)

> **NOTE**: An `unlock key` will be returned, which can be used to unlock the API for that namespace. 
> Preserve this key to unlock the API in the future!
```
vault namespace lock
```


### Unlock
[Docs](https://www.vaultproject.io/docs/concepts/namespace-api-lock#unlocking)

> **NOTE**: 
> 1) In general, an `unlock key` is required to unlock the API. 
> This is the same as the `unlock key` provided when the namespace was locked.
> 
> 2) The `unlock key` requirement can be overriden by using a `root token` with the unlock request.
```
vault namespace unlock
```


## Authentication

[Docs: Higlevel](https://www.vaultproject.io/docs/concepts/auth)

[Docs: Full](https://www.vaultproject.io/docs/auth)

Vault supports multiple auth methods simultaneously.

> **NOTE**: Authentication works by verifying your identity and then generating a `token` to associate with that identity.
> For example, even though you may authenticate using something like `GitHub`, Vault generates a unique access `token` for you to use for future requests.


### General

List configured auth methods
```
vault auth list

      Path               Type        Accessor                  Description
      ----               ----        --------                  -----------
      token/             token       auth_token_e34b827e       token based credentials
      xbs_first_auth/    userpass    auth_userpass_f751c1c1    n/a
```

Describe auth method that was used to create specific authentication (for example, `auth/xbs_first_auth`)
```
vault path-help auth/xbs_first_auth
```



### Userpass (basic auth)

Create
```
vault write sys/auth/xbs_first_auth type=userpass
```

### AppRole
[Example 1: Via curl](https://learn.hashicorp.com/tutorials/vault/getting-started-apis?in=vault/getting-started)

### Token

> **NOTE**: Here is only info how to use token you already have.
> Read appropriate block to get more info about `tokens` as core method for authentication within Vault

Authenticate
```
# CLI
vault login token=<token>

      # OR
      
# API: Option 1
curl \
    --header "Authorization: Bearer <INSERT_VAULT_TOKEN_HERE>" \
    --request LIST \
    http://127.0.0.1:8200/v1/auth/token/accessors
    
      # OR
      
# API: Option 2
curl \
    --header "X-Vault-Token: <INSERT_VAULT_TOKEN_HERE>" \
    --request LIST \
    http://127.0.0.1:8200/v1/auth/token/accessors
```

### GitHub

Authenticate
```
vault login -method=github token=<token>

      # OR
      
vault login -method=github
```



## Tokens

[Docs: Highlevel](https://www.vaultproject.io/docs/concepts/tokens#tokens)
[Docs: Full](https://www.vaultproject.io/docs/auth/token)
[Docs: API](https://www.vaultproject.io/api/auth/token)



> **NOTE**: Since tokens are considered opaque values, their structure is undocumented and subject to change.

`Tokens` are the core method for authentication within Vault. 

`Tokens` can be used directly or `auth method`s can be used to dynamically generate `tokens` based on external identities.



## Policy

### General

List all policies
```
vault policy list
```

List system policies (?)
```
vault read sys/policy

   # OR

curl \
  --header "X-Vault-Token: ..." \
  https://vault.hashicorp.rocks/v1/sys/policy
```

Import policy from HCL file
```
# HCL file example
#
# path "secret/dev/team-1/*" {
#   capabilities = ["create", "update", "read"]
# }
# path "secret/dev/global" {
#   capabilities = ["read"]
# }
#
# vault policy write <POLICY_NAME> <HCL_FILE_PATH>

vault policy write dev-team-1 team-one-policy.hcl
```

Assign a policy to a token
```
vault token create -policy=dev-team-1
```
### Built-in Policies

Vault has two built-in policies: `default` and `root`


### Default policy
The `default` policy is a built-in Vault policy that cannot be removed. 

By default, it is attached to all tokens, but may be explicitly excluded at token creation time by supporting authentication methods.

To view all permissions granted by the default policy on your Vault installation, run:
```
vault read sys/policy/default
```

To disable attachment of the default policy:
```
vault token create -no-default-policy

   # OR
   
curl \
  --request POST \
  --header "X-Vault-Token: ..." \
  --data '{"no_default_policy": "true"}' \
  https://vault.hashicorp.rocks/v1/auth/token/create  
```


### Root Policy

The `root` policy is a built-in Vault policy that can not be modified or removed. Any user associated with this policy becomes a root user. A root user can do **anything** within Vault. As such, it is **highly recommended** that you revoke any root tokens before running Vault in production.

> **NOTE**: When a Vault server is first initialized, there always exists one root user. This user is used to do the initial configuration and setup of Vault. After configured, the initial root token should be revoked and more strictly controlled users and authentication should be used.

To revoke a root token, run:
```
vault token revoke "<token>"

   # OR
   
curl \
  --request POST \
  --header "X-Vault-Token: ..." \
  --data '{"token": "<token>"}' \
  https://vault.hashicorp.rocks/v1/auth/token/revoke
```


## Lease

[Docs: General](https://www.vaultproject.io/docs/concepts/lease)
[Docs: Available Parameters](https://www.vaultproject.io/api-docs/system/leases)

> **NOTE**: With every dynamic secret and service type authentication token, Vault creates a lease: metadata containing information such as a time duration, renewability, and more. 
> Vault promises that the data will be valid for the given duration, or Time To Live (TTL). 
> Once the lease is expired, Vault can automatically revoke the data, and the consumer of the secret can no longer be certain that it is valid.


> The benefit should be clear: consumers of secrets need to check in with Vault routinely to either renew the lease (if allowed) or request a replacement secret. This makes the Vault audit logs more valuable and also makes key rolling a lot easier.


> In addition to renewals, a lease can be revoked. When a lease is revoked, it invalidates that secret immediately and prevents any further renewals. 

> When a token is revoked, Vault will revoke all leases that were created using that token.


### List

All types of leases:
```
vault list sys/leases/lookup/auth/approle/login

      Keys
      ----
      auth/
```

The leases for `auth` type:
```
vault list sys/leases/lookup/auth

      Keys
      ----
      auth/
```

The leases for `approle` `auth` type:
```
vault list sys/leases/lookup/auth/approle

      Keys
      ----
      login/
```

The leases for logins of `approle` `auth` type:
```
vault list sys/leases/lookup/auth/approle/login

      Keys
      ----
      h0c817b0d54735abfe5e12ed6c01d56fb78e755584e3934f360bad6bd383415c0
      h0e6113599dd06f58ee7e954cbaab6b901c4c85a0b3ed484a5096cdfa700eccb1
```

### Describe

Describe specific Lease
```
# Remember the ID of the necessary Lease
# (this is everything under `sys/leases/lookup` mount)
export LEASE_ID="auth/approle/login/h457d2cc67e6901b48b9bd665f6ad4abb67e47c21eee60bfb33c2b491fea9e3b3"

# Describe specified Lease
vault lease lookup ${LEASE_ID}

      Key             Value
      ---             -----
      expire_time     2022-01-04T17:58:39.4583564Z
      id              auth/approle/login/h457d2cc67e6901b48b9bd665f6ad4abb67e47c21eee60bfb33c2b491fea9e3b3
      issue_time      2022-01-04T16:58:39.458377Z
      last_renewal    <nil>
      renewable       true
      ttl             16m27s
```

### Revoke

```
# Remember the ID of the necessary Lease
# (this is everything under `sys/leases/lookup` mount)
export LEASE_ID="auth/approle/login/h21e043a571e5b6ade3d5389c43f7fbf4f6d77bb25e415e43612b170de9109f5f"

# Revoke it
vault lease revoke ${LEASE_ID}
```






