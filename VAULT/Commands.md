
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
# Single secret
vault kv put secret/xbs_first_secret_via_cli xbs_key_name=xbs_key_value

   # OR

# Multiply secrets
vault kv put secret/xbs_first_secret_via_cli ttl='30s' username='xbs_username' password='xbs_pwd'
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

#### Enable

Enable `approle` auth method by mounting its endpoint at `/sys/auth/approle`
```
vault auth enable approle

   # OR
   
curl --header "X-Vault-Token: $VAULT_TOKEN" \
    --request POST \
    --data '{"type": "approle"}' \
    $VAULT_ADDR/v1/sys/auth/approle
```

#### List

See created `jenkins-role` role
```
vault read auth/approle/role/jenkins-role
```


#### Create role

Create a role named `jenkins-role` with `jenkins-policy` policy attached.
```
# To attach multiple policies, pass the policy names as a comma separated string: 
# token_policies="jenkins,anotherpolicy"

vault write auth/approle/role/jenkins-role token_policies="jenkins-policy" \
    token_ttl=1h token_max_ttl=4h
```

#### Get `RoleID`

Retrieve the `RoleID` for the `jenkins-role` role
```
# The `auth/approle/role/<ROLE_NAME>/role-id` endpoint provides `RoleID` (constant one for specific `Role`);

# The constant one, randomly generated 
vault read auth/approle/role/jenkins-role/role-id

   # OR
   
# The constant one, which was customly defined
#   Define
vault write auth/approle/role/jenkins-role/role-id role_id=xbs-markets-live01
#   Read
vault read auth/approle/role/jenkins-role/role-id
```

#### Get `SecretID`

[Docs: Parameters for the SecretID](https://www.vaultproject.io/api/auth/approle#generate-new-secret-id)
[Docs: Response Wrapping](https://learn.hashicorp.com/tutorials/vault/approle#response-wrap-the-secretid)

##### Wrapped

Instead of generating and getting of `SecretID` itself, response wrapping could be used insted (to get a link to the secret id instead of secret itself)
```
vault write -wrap-ttl=360s -force auth/approle/role/jenkins-role/secret-id

  Key                              Value
  ---                              -----
  wrapping_token:                  s.BW3oUqDf7iGg1x96fccfX5vp
  wrapping_accessor:               RLzjMCeIR7WbdXcdHO71Bl24
  wrapping_token_ttl:              6m
  wrapping_token_creation_time:    2022-01-04 16:49:24.0887075 +0000 UTC
  wrapping_token_creation_path:    auth/approle/role/jenkins-role/secret-id
  wrapped_accessor:                f412d75c-e5da-a1fe-c599-1e17cf96cc37
```

Application then can use this `wrapping_token` to get real `SecretID`

> **NOTE**: App should know, where to find Vault (via `VAULT_ADDR` var, for example) 
```
VAULT_TOKEN="s.BW3oUqDf7iGg1x96fccfX5vp" vault unwrap

  Key                   Value
  ---                   -----
  secret_id             2f34adc9-5542-61af-63eb-bed459b6182a
  secret_id_accessor    f412d75c-e5da-a1fe-c599-1e17cf96cc37
  secret_id_ttl         0s
```

##### Not wrapped (not recommended)
Generate a `SecretID` for the `jenkins-role` role
```
# The `auth/approle/role/<ROLE_NAME>/secret-id` endpoint generates a new `SecretID` 
# (new one after every generation for the same `RoleID`, with some lifetime paramaters that were set up during `Role` creation).

# The `-force` (or `-f`) flag forces the write operation to continue without any data values specified. 
# Or you can set parameters such as `cidr_list`.

# If you specified `secret_id_ttl`, `secret_id_num_uses`, or `bound_cidr_list` on the step of Role creation,
# the generated `SecretID` carries out the conditions.


# Generate a randome one
vault write -force auth/approle/role/jenkins-role/secret-id

   # OR
   
# Set custom one (thus, static)
vault write auth/approle/role/jenkins-role/custom-secret-id secret_id=xbs-pwd-olala-who-knows-it
```



#### Login with `RoleID` & `SecretID`

> Persona: `application`

The client (in this case, _Jenkins_) uses the `RoleID` and `SecretID` passed by the `admin` to authenticate with _Vault_.


To **generate** a new token for furhter login, use the `auth/approle/login` endpoint by passing the `RoleID` and `SecretID`.
```
vault write auth/approle/login role_id="5be0c647-7d92-7b29-63fa-f1fe27ad9169" \
    secret_id="2f34adc9-5542-61af-63eb-bed459b6182a"
    
  Key                     Value
  ---                     -----
  token                   s.u3vlFq406EvWwcbcpTjz1Qya
  token_accessor          7cnZWtgz7xRQ6zKdipktLNIE
  token_duration          1h
  token_renewable         true
  token_policies          ["default" "jenkins-policy"]
  identity_policies       []
  policies                ["default" "jenkins-policy"]
  token_meta_role_name    jenkins-role
```

For example, store the generated token value in an environment variable named, `APP_TOKEN`.
```
export APP_TOKEN="s.u3vlFq406EvWwcbcpTjz1Qya"
```


#### Read secrets using the `AppRole` token

> Persona: `application`

Once receiving a token from Vault, the client can make future requests using this token.

Verify that you can access the secrets at necessary `secret/mysql/webapp` (secrets, that were created initially for the application).

##### Via CLI
```
VAULT_TOKEN=$APP_TOKEN vault kv get secret/mysql/webapp

  ====== Metadata ======
  Key              Value
  ---              -----
  created_time     2021-06-08T02:34:23.182299Z
  deletion_time    n/a
  destroyed        false
  version          1

  ====== Data ======
  Key         Value
  ---         -----
  db_name     users
  password    passw0rd
  username    admin
```

##### Via API

```
curl --header "X-Vault-Token: $APP_TOKEN" \
    $VAULT_ADDR/v1/secret/mysql/webapp | jq -r ".data"

  {
    "data": {
      "db_name": "users",
      "password": "passw0rd",
      "username": "admin"
    },
    "metadata": {
      "created_time": "2022-01-04T15:20:04.1790576Z",
      "custom_metadata": null,
      "deletion_time": "",
      "destroyed": false,
      "version": 1
    }
  }
```




### AWS

Enable
```
vault auth enable aws
```

Create `dev-role-iam` Vault role, that allows to auth with `burt-temporary-tests-vault-vault-client-role` AWS IAM role to provide `myapp` Vault policy
```
vault write auth/aws/role/dev-role-iam auth_type=iam bound_iam_principal_arn="arn:aws:iam::883994838493:role/burt-temporary-tests-vault-vault-client-role" policies=myapp ttl=24h
```


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

### List

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

### Read

Read `myapp` policy
```
vault policy read myapp
```

### Write

Write policy from a `HCL` file
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






