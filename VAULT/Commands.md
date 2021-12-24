
## Start/Stop

### "Dev" mode

This dev-mode server requires no further setup, and your local vault CLI will be authenticated to talk to it.
Every feature of Vault is available in "dev" mode. 
The `-dev` flag just short-circuits a lot of setup to insecure defaults.
> **NOTE**: Never, ever, ever run a "dev" mode server in production. It is insecure and will lose data on every restart (since it stores data in-memory). It is only made for development or experimentation.
```
vault server -dev
```





