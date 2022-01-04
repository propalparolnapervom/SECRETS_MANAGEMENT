## Challenge 
How to use `AppRole` auth method to get access to the `Vault` secrets for the application (`Jenkins`, for example.


## Docs

[Docs: Configuration](https://www.vaultproject.io/docs/auth/approle#configuration)
[Docs: AppRole tutorial](https://learn.hashicorp.com/tutorials/vault/approle)



## Prerequisites

The `Vault` instance is up and running, initialized and unsealed. For example:
```
docker run -p 8200:8200 vault

  Unseal Key: bYPxDOJ6P1Eo8eY/SDgslo9iN/BA4/vtpTK0MSZNIqw=
  Root Token: s.mbzQqfz0YwZykCtJL5N2zgNe

export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN="s.mbzQqfz0YwZykCtJL5N2zgNe"
```

Create some test secrets to be available by the app:
```
# Create
vault kv put secret/mysql/webapp db_name="users" username="admin" password="passw0rd"

# Read
vault kv get secret/mysql/webapp
```


## Step 1: Enable AppRole auth method

> Persona: `admin`

Enable `approle` auth method by mounting its endpoint at `/sys/auth/approle`
```
vault auth enable approle

   # OR
   
curl --header "X-Vault-Token: $VAULT_TOKEN" \
    --request POST \
    --data '{"type": "approle"}' \
    $VAULT_ADDR/v1/sys/auth/approle
```



## Step 2: Create a role with policy attached

> Persona: `admin`

In this example, you are going to create a role for the app persona (`jenkins` in our scenario).


### 2.1. Via CLI

#### 2.1.1. Policy

Create an policy named `jenkins-policy`.
```
vault policy write jenkins-policy -<<EOF
# Read-only permission on secrets stored at 'secret/data/mysql/webapp'
path "secret/data/mysql/webapp" {
  capabilities = [ "read" ]
}
EOF
```

#### 2.1.2. Role

[Docs: Parameters for the Role](https://www.vaultproject.io/api-docs/auth/approle#create-update-approle)

Create a role named `jenkins-role` with `jenkins` policy attached.
```
# To attach multiple policies, pass the policy names as a comma separated string: 
# token_policies="jenkins,anotherpolicy"

vault write auth/approle/role/jenkins-role token_policies="jenkins-policy" \
    token_ttl=1h token_max_ttl=4h
```

See created role
```
vault read auth/approle/role/jenkins-role
```



## Step 3.1: Get `RoleID`

Retrieve the `RoleID` for the `jenkins-role` role
```
# The `auth/approle/role/<ROLE_NAME>/role-id` endpoint provides `RoleID` (constant one for specific `Role`);

vault read auth/approle/role/jenkins-role/role-id
```

## Step 3.2: Get `SecretID`
[Docs: Parameters for the SecretID](https://www.vaultproject.io/api/auth/approle#generate-new-secret-id)
[Docs: Response Wrapping](https://learn.hashicorp.com/tutorials/vault/approle#response-wrap-the-secretid)

### Wrapped

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
```
VAULT_TOKEN="s.BW3oUqDf7iGg1x96fccfX5vp" vault unwrap

  Key                   Value
  ---                   -----
  secret_id             2f34adc9-5542-61af-63eb-bed459b6182a
  secret_id_accessor    f412d75c-e5da-a1fe-c599-1e17cf96cc37
  secret_id_ttl         0s
```

### Not wrapped (not recommended)
Generate a `SecretID` for the `jenkins-role` role
```
# The `auth/approle/role/<ROLE_NAME>/secret-id` endpoint generates a new `SecretID` 
# (new one after every generation for the same `RoleID`, with some lifetime paramaters that were set up during `Role` creation).

# The `-force` (or `-f`) flag forces the write operation to continue without any data values specified. 
# Or you can set parameters such as `cidr_list`.

# If you specified `secret_id_ttl`, `secret_id_num_uses`, or `bound_cidr_list` on the step of Role creation,
# the generated `SecretID` carries out the conditions.

vault write -force auth/approle/role/jenkins-role/secret-id
```



## Step 4: Login with RoleID & SecretID

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


## Step 5: Read secrets using the AppRole token

> Persona: `application`

Once receiving a token from Vault, the client can make future requests using this token.

Verify that you can access the secrets at necessary `secret/mysql/webapp` (secrets, that were created initially for the application).

### Via CLI
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

### Via API

```
curl --header "X-Vault-Token: $APP_TOKEN" \
    $VAULT_ADDR/v1/secret/data/mysql/webapp | jq -r ".data"

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





















