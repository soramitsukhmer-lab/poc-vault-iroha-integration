# About

This repository explains how to use HashiCorp Vault with a customized `transit` secrets engine to support `ed25519sha3` key generation and signing.

Please refer to the source code for the implementation details at [soramitsukhmer-lab/vault#builtin/logical/transit/ed25519sha3](https://github.com/soramitsukhmer-lab/vault/tree/builtin/logical/transit/ed25519sha3) and [soramitsukhmer-lab/go-ed25519sha3](https://github.com/soramitsukhmer-lab/go-ed25519sha3).

## Documentation

The official `transit` secret engine and `iroha-transit` secret engine is not that different from each other. You can follow the official [Transit secrets engine (API)
](https://developer.hashicorp.com/vault/api-docs/secret/transit#transit-secrets-engine-api) documentation to implementing features for  `iroha-transit` secret engine.

The only different is `iroha-transit` implemented `ed25519-sha3-512` key type, while the official one only support `ed25519`.

```bash
# Enable the transit secrets engine and create a key
vault secrets enable iroha-transit
vault write iroha-transit/keys/iroha type=ed25519-sha3-512
```

## Starting Vault

Start a Vault server in development mode using Docker. This will run Vault in a single process with an in-memory storage backend, suitable for testing and development purposes.
```bash
docker run --rm -it -p 8200:8200 harbor.sorakh.io/soramitsukhmer-lab/vault:dev server -dev -dev-root-token-id=toor
```

Setup the Vault environment variables to point to the running Vault server:
```bash
export VAULT_ADDR=http://localhost:8200
export VAULT_TOKEN=toor
```

Source for the container image can be found at [soramitsukhmer-lab/docker-vault](https://github.com/soramitsukhmer-lab/docker-vault).

## How to use the `transit` secrets engine

Enable the `iroha-transit` secrets engine and create a key named `iroha` with the `ed25519-sha3-512` type. The key will be exportable, allowing you to retrieve the public key for use in signing and verification operations.

```bash
# Enable the transit secrets engine and create a key
vault secrets enable iroha-transit
vault write iroha-transit/keys/iroha type=ed25519-sha3-512

# Export the public key
vault read -format=json iroha-transit/export/public-key/iroha
# or
vault read -format=json iroha-transit/export/public-key/iroha  | jq -r '.data.keys["1"]'
```

Signing and verification can be done using the following commands:
```bash
# Sign a message
payload='{
    "input": "KvJOYGLq+NU3U7ZLU2Yu0EJob9sa8otlZrFTfo8etNk="
}'
curl \
    --header "X-Vault-Token: toor" \
    --request POST \
    --data "${payload}" \
    http://127.0.0.1:8200/v1/iroha-transit/sign/iroha
```

```bash
# Verify a signature
payload='{
    "input": "KvJOYGLq+NU3U7ZLU2Yu0EJob9sa8otlZrFTfo8etNk=",
    "signature": "vault:v1:xzBH891iWVXe8YvfVN5ZHvDYCFHDAXAIq5O9vEuJjsn2BJ/D4nvHfhqcvjJiNwhPPKUObFT7t9G6FmA3kMFgDw=="
}'
curl \
    --header "X-Vault-Token: toor" \
    --request POST \
    --data "${payload}" \
    http://127.0.0.1:8200/v1/iroha-transit/verify/iroha
```

## Iroha Java SDK that Support Vault
Please add this `iroha-java` sdk dependency in `build.gradle.kts` in order for testing integration poc:
```
repositories {
    // Other repositories
    maven {
        name = "GitHubSoramitsuKhmerApacheMavenPackages"
        url = uri("https://maven.pkg.github.com/soramitsukhmer/*")
        credentials {
            username = System.getenv("GIT_PUBLISH_USER")
            password = System.getenv("GIT_PUBLISH_PASSWORD")
        }
    }
    // Other repositories
}
dependencies {
    // Other dependencies
    implementation("com.soramitsukhmer.iroha-java:client:7.0.1-rc.4")
    // Other dependencies
}
```
Please check sdk dependency repository can be found at [soramitsukhmer/iroha-java](https://github.com/soramitsukhmer/iroha-java)
