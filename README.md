# LabOps Dockerized Hashi Vault
## Current Documentation Version:
- Latest Update Version 1.0.0, updated by Matthew Branche on 09/07/2022
## Shell Purpose and Introduction
The purpose of this shell, is to provide users with a central vault for proper credential storage, while allowing automation api and user access via authorized tokens. LDAP not configured yet. LDAP Slated for 2.0.0

## Architecture
This Shell uses a Docker Compose functionality and uses the below directory architecture:
- Main vault directory
    - main files:
      - changelog.md (documentation of release notes for each version upgrade of the shell)
      - README.md 
      - version.md (file used to document current version)
      - docker-compose.yml (primary docker-compose file to build vault)
      - consul
        - primary consul directory - configured automatically
      - vault
        - Dockerfile (primary docker file used to build the vault container image)
        - config (set to persistent via docker-compose.yml)
          - consul-config.json
          - vault-config.json
        - data
          - primary data directory (set to persistent via docker-compose.yml)
        - logs
          - primary logs directory (set to persistent via docker-compose.yml)
        - policies
          - primary policies directory (set to persistent via docker-compose.yml)
          - app-policy.json (primary policy file)

## Initialization of the vault container
- Validate your desired linux box has the following loaded: docker, docker-compose, make
  - Highly suggested to use Ubuntu22 or higher (other linux OS's can be used, but that was what this shell was initially deployed on)
  - if not installed, please follow the install reference docs in the reference docs section
- Download a copy of the labops vault from Git to the local linux box (or git clone to your desired location)
- cd into the shell directory
- update the .env file, and the VAULT_ADDR variable with your server's fqdn (www.example.com is provided as an example)
- run the following command to start the server:
```
$ docker-compose up -d --build
```
## Configuration and Deployment of the Vault
- Access the container with the following:
```
$ docker-compose exec vault bash
... (from inside the container)

# vault operator init
```
- This should dump out 5 unseal encryption keys, and your root token. Copy all unseal keys, and your root token out, and save for later.
- example output:
```
### BELOW ARE EXAMPLE KEYS FOR YOU TO SEE POTENTIAL OUTPUT - NOT FOR USE IN DEPLOYED INSTANCES ###
bash-4.4# vault operator init
Unseal Key 1: p9ySCyRaHXUhQQAw3PgkQhSvoe+mexRlZGILDi2ieLji
Unseal Key 2: a8IQbNnbUWLx5mK//nkG0NIO4XtYbeqOFnS7R1STJhQg
Unseal Key 3: 4Q+qgYNYTRqER5NdrzFwyYSmI6ZcQ4qYcvat0YKCGqJB
Unseal Key 4: AlXy7LO1Nyxxp5GAVYe2MEusk8chXJb/q0rnT5hxOsB6
Unseal Key 5: XHtoC4husOQCwXNEyT4Du6D9zF6T2UJgEWVRoCC/62kr

Initial Root Token: 213d7dad-c2fd-de83-61bf-bcc9ab43d480

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 3 key to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```
- Take note of the unseal keys and the initial root token. We will need to provide three of the unseal keys every time the Vault server is re-sealed or restarted.
- provide these, like the below example output:
```
bash-4.4# vault operator unseal 4Q+qgYNYTRqER5NdrzFwyYSmI6ZcQ4qYcvat0YKCGqJB
Key             Value
---             -----
Seal Type       shamir
Sealed          true
Total Shares    5
Threshold       3
Version         0.10.3
Cluster Name    vault-cluster-7358692f
Cluster ID      4a546e14-2781-6057-a9f6-46cce8f34940
HA Enabled      false

bash-4.4# vault operator unseal AlXy7LO1Nyxxp5GAVYe2MEusk8chXJb/q0rnT5hxOsB6
Key             Value
---             -----
Seal Type       shamir
Sealed          true
Total Shares    5
Threshold       3
Version         0.10.3
Cluster Name    vault-cluster-7358692f
Cluster ID      4a546e14-2781-6057-a9f6-46cce8f34940
HA Enabled      false

bash-4.4# vault operator unseal 
Unseal Key (will be hidden): 
Key             Value
---             -----
Seal Type       shamir
Sealed          false
Total Shares    5
Threshold       3
Version         0.10.3
Cluster Name    vault-cluster-7358692f
Cluster ID      4a546e14-2781-6057-a9f6-46cce8f34940
HA Enabled      false
```
- finally we can authenticate using the root token and set advanced logging
```
bash-4.4# vault login 213d7dad-c2fd-de83-61bf-bcc9ab43d480
bash-4.4# vault audit enable file file_path=/vault/logs/audit.log
```
- You are now all set to login your server at http://www.example.com:8200/

## Docker Environment Variables
file: .env
- Variables:
    - VAULT_ADDR: used to provide the server with your dns name of the server, to be able to offer services
```
VAULT_ADDR=http://www.example.com:8200

```
## Supportability
- for supportability or questions, please create an ESM ticket or email the IEO West LabOps team: matthew.branche@dell.com

## Reference docs
- installing vault with docker-compose reference: https://www.bogotobogo.com/DevOps/Docker/Docker-Vault-Consul.php
- education on hashi vault, vault credential storage, and api token authentication: https://learn.hashicorp.com/collections/vault/cloud?utm_source=google&utm_medium=sem&utm_campaign=na_search_brand_hcp_vault&utm_content=17089930765-133042353981-595455078180&utm_offer=signup&gclid=Cj0KCQjwyOuYBhCGARIsAIdGQRNSkOaSZTIcT21EAfNmIDf-LkVc-_o7elN3zfvkNjPrEZ14-6GtkpMaAnf2EALw_wcB
- installing docker reference: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04
- installing docker-compose reference: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-22-04

## Docker and Docker-compose information
- For specifics on the Dockerfile, and docker-compose.yml functionality, please contact matthew.branche@dell.com

## License
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
