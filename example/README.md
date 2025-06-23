This is an example `compose.yml` file for running a Vault server in development mode with a custom `os-shell` container for managing Vault plugin catalog.

The compose file uses the official Vault image and a special `os-shell` image that provides a shell environment for executing commands related to Vault's plugin catalog from [soramitsukhmer-lab/os-shell](https://github.com/soramitsukhmer-lab/os-shell).

To run the Vault server and the shell container, use the following command:

```bash
docker compose up -d
```

Then you can access the shell container to manage the Vault plugin catalog:

```bash
docker exec -it os-shell /bin/bash

# Once inside the shell, the following commands can be used to manage the Vault plugin catalog:

# Download or update plugin catalog
vault-plugin-catalog update

# List available plugins
vault-plugin-catalog list

# Download and install the plugin to appropriate directory
vault-plugin-catalog install iroha-transit

# This will not register the plugin to Vault but will print all necessary command to register the plugin
vault-plugin-catalog register iroha-transit
```
