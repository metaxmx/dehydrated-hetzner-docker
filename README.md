# dehydrated-hetzner-docker

Docker container for dehydrated ACME Client with Hetzner DNS Client.

## References

On Docker-Hub: https://hub.docker.com/r/metax/dehydrated-hetzner

Base Docker Image: `debian:stretch`

Dehydrated ACME ("Let's Encrypt") Client by @lukas2511: https://github.com/lukas2511/dehydrated

Hetzner Dehydrated Hook by @rembik: https://github.com/rembik/dehydrated-hetzner-hook

## Example config and usage

**`/opt/dehydrated/config`**:
```
CHALLENGETYPE="dns-01"
CONTACT_EMAIL="mail@mydomain.com"
HOOK="${BASEDIR}/hooks/hetzner/hook.py"
```

**`/opt/dehydrated/domains.txt`**:
```
mydomain.com *.mydomain.com
myotherdomain.net *.myotherdomain.net
```

**`/opt/dehydrated/secrets.env`**:
```
HETZNER_AUTH_USERNAME=myhetznerusername
HETZNER_AUTH_PASSWORD=*****************
```

**`/usr/local/bin/run-dehydrated.sh`**:
```bash
#!/bin/bash

CONTAINER_NAME="dehydrated"
DOCKER_HOSTNAME="$CONTAINER_NAME"
IMAGE="metax/dehydrated-hetzner"
BASE_DIR="/opt/dehydrated"

ENV_FILE="$BASE_DIR/secrets.env"

CERTS_DIR="$BASE_DIR/certs"
ACCOUNTS_DIR="$BASE_DIR/accounts"
CHALLENGE_DIR="$BASE_DIR/challenge"
CONFIG_FILE="$BASE_DIR/config"
DOMAINS_FILE="$BASE_DIR/domains.txt"

docker run --rm -it \
  --name $CONTAINER_NAME \
  -v $CERTS_DIR:/etc/dehydrated/certs \
  -v $ACCOUNTS_DIR:/etc/dehydrated/accounts \
  -v $CHALLENGE_DIR:/etc/dehydrated/challenge \
  -v $CONFIG_FILE:/etc/dehydrated/config:ro \
  -v $DOMAINS_FILE:/etc/dehydrated/domains.txt:ro \
  --env-file ${ENV_FILE} \
  -h $DOCKER_HOSTNAME \
  "$IMAGE" \
  "$@"
```

### Run dehydrated:

Once to create directory structure:  
`mkdir -p /opt/dehydrated/{accounts,certs,challenge}`

Once to created Let's Encrypt account:  
`run-dehydrated --register --accept-terms`

Cron (Create / Refresh certificates in `domains.txt`):  
`run-dehydrated -c`

Force refresh:  
`run-dehydrated -c --force`

### Use certificates

* `/opt/dehydrated/certs/<mydomain>/privkey.pem`: Private Key
* `/opt/dehydrated/certs/<mydomain>/cert.pem`: X.509 Certificate
* `/opt/dehydrated/certs/<mydomain>/chain.pem`: Root Certificate (and optional intermediate certificates)
* `/opt/dehydrated/certs/<mydomain>/fullchain.pem`: X.509 Certificate including chain (root certificate)
