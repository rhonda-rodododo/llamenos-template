# Llámenos

<!-- metadata -->

* **Category**: Communication
* **Status**: 1, alpha
* **Image**: [`ghcr.io/rhonda-rodododo/llamenos`](https://github.com/rhonda-rodododo/llamenos/pkgs/container/llamenos), upstream
* **Healthcheck**: Yes
* **Backups**: Yes
* **Email**: No
* **Tests**: No
* **SSO**: No

<!-- endmetadata -->

Secure, open-source crisis response hotline software with end-to-end encryption. Callers dial a phone number; calls are routed to on-shift volunteers via parallel ringing. Volunteers log encrypted notes in a web app. Admins manage shifts, volunteers, and ban lists.

* **Upstream**: <https://github.com/rhonda-rodododo/llamenos>
* **Docs**: <https://llamenos-hotline.com/docs/deploy-coopcloud>

## Features

| Feature | Status |
|---------|--------|
| E2EE notes & transcriptions | Yes |
| Multi-provider telephony (Twilio, SignalWire, Vonage, Plivo, Asterisk) | Yes |
| Multi-channel messaging (SMS, WhatsApp, Signal) | Yes |
| Encrypted reporting portal | Yes |
| Multi-language (13 languages) | Yes |
| Admin/volunteer role separation | Yes |
| Docker Swarm secrets | Yes |
| Backupbot integration | Yes |

## Quick start

```bash
# Add your server (if not already added)
abra server add hotline.example.com

# Create a new app
abra app new llamenos --server hotline.example.com --domain hotline.example.com

# Generate all secrets
abra app secret generate -a hotline.example.com

# Deploy
abra app deploy hotline.example.com
```

Visit `https://hotline.example.com` and follow the setup wizard to create your admin account.

## Services

The recipe deploys five core services:

| Service | Image | Purpose |
|---------|-------|---------
| **web** | `nginx:1.27-alpine` | Reverse proxy with Traefik labels |
| **app** | `ghcr.io/rhonda-rodododo/llamenos` | Node.js application server |
| **db** | `postgres:17-alpine` | PostgreSQL database |
| **minio** | `minio/minio` | S3-compatible file storage |
| **relay** | `dockurr/strfry` | Nostr relay for real-time events |

## Secrets

All secrets are managed via Docker Swarm secrets (versioned, immutable):

| Name | Type | Description |
|------|------|-------------|
| `hmac_secret` | hex (64 chars) | HMAC signing key for session tokens |
| `server_nostr` | hex (64 chars) | Server Nostr identity key |
| `db_password` | alnum (32 chars) | PostgreSQL password |
| `minio_access` | alnum (20 chars) | MinIO access key |
| `minio_secret` | alnum (40 chars) | MinIO secret key |

Generate all at once:

```bash
abra app secret generate -a hotline.example.com
```

> To rotate a secret, bump its version in `abra app config` (e.g., `SECRET_HMAC_SECRET_VERSION=v2`), regenerate, and redeploy.

## First login

After deployment, visit your domain in a browser. The setup wizard guides you through:

1. **Create admin account** — generates a cryptographic keypair in your browser
2. **Name your hotline** — set the display name
3. **Choose channels** — enable Voice, SMS, WhatsApp, Signal, and/or Reports
4. **Configure providers** — enter credentials for each channel
5. **Review and finish**

## Optional services

Enable optional services by adding overlays to `COMPOSE_FILE` in your app config:

* **Transcription** (4 GB+ RAM required):
  ```
  COMPOSE_FILE=compose.yml:compose.transcription.yml
  ```

* **Asterisk PBX** (self-hosted SIP telephony):
  ```
  COMPOSE_FILE=compose.yml:compose.asterisk.yml
  ```
  Additional secrets required: `abra app secret generate hotline.example.com ari_password bridge_secret`

* **Signal messaging**:
  ```
  COMPOSE_FILE=compose.yml:compose.signal.yml
  ```

* **Multiple overlays**:
  ```
  COMPOSE_FILE=compose.yml:compose.transcription.yml:compose.signal.yml
  ```

## Updating

```bash
abra app upgrade hotline.example.com
```

Data is persisted in Docker volumes and survives upgrades.

## Backups

The recipe includes [backupbot](https://docs.coopcloud.tech/backupbot/) labels for automated PostgreSQL and MinIO backups. If your server runs backupbot, backups happen automatically.

For manual backups, use the included script:

```bash
./pg_backup.sh <stack-name>
./pg_backup.sh <stack-name> /backups    # custom directory, 7-day retention
```
