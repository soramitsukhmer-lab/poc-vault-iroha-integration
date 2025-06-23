This is an example `compose.yml` file for running a Vault server in development mode with a custom `os-shell` container for managing Vault plugin catalog.

The compose file uses the official Vault image and a special `os-shell` image that provides a shell environment for executing commands related to Vault's plugin catalog from [soramitsukhmer-lab/os-shell](https://github.com/soramitsukhmer-lab/os-shell).
