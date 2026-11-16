<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/icon-dark.svg">
    <img src="assets/icon.svg" alt="Vaultwarden logo" width="128">
  </picture>
</p>

<h1 align="center">aac-vaultwarden</h1>

<p align="center">Self-hosted deployment definition for Vaultwarden, a Bitwarden-compatible password manager server.</p>

<p align="center">
  <a href="https://github.com/dani-garcia/vaultwarden"><img alt="Upstream" src="https://img.shields.io/badge/upstream-dani--garcia%2Fvaultwarden-blue?logo=github"></a>
  <img alt="Image" src="https://img.shields.io/badge/image-vaultwarden%2Fserver%3A1.36.0-2496ED?logo=docker&logoColor=white">
  <a href="https://github.com/dani-garcia/vaultwarden/blob/main/LICENSE.txt"><img alt="License" src="https://img.shields.io/badge/license-AGPL--3.0-green"></a>
</p>

## About

Vaultwarden is an unofficial, Bitwarden-compatible server written in Rust. It works with the
official Bitwarden clients (browser extensions, desktop and mobile apps, CLI), so passwords are
stored and synchronized in a server you run yourself. It is much lighter than the official
Bitwarden server and can run on small hardware.

Key features (as provided by upstream):

- Compatible with the official Bitwarden clients
- Web vault included
- Optional PostgreSQL backend (this deployment uses PostgreSQL 18)
- Real-time sync via WebSocket notifications
- Single sign-on via OpenID Connect (this deployment is prepared for SSO)
- Attachments, Send and two-factor authentication support

Vaultwarden is not affiliated with or endorsed by Bitwarden, Inc.

## Upstream

- Project: <https://github.com/dani-garcia/vaultwarden>
- Container image: `vaultwarden/server:1.36.0` (Docker Hub)
- Database: `postgres:18` (deployed as `aac-vaultwarden-db`)
- License: AGPL-3.0 (upstream project)

## Quick start (Docker Compose)

The Compose file is rendered from `service.yml` by the App-as-Code template engine. For a
manual run, the external networks used for routing must exist first:

```bash
docker network create services-secured
docker network create services-exposed

# Provide the values listed under "Configuration" in .env / stack.env
docker compose up -d
```

Secrets are not stored in this repository. Provide them through the deployment pipeline
or create the `.env` and `stack.env` files yourself (see `example.env` for all variables).
Do not commit them.

## Configuration

Environment variables (defaults from `service.yml`; `CHANGE_ME` must be set per deployment):

| Variable | Default | Description |
|---|---|---|
| `ADMIN_TOKEN` | `CHANGE_ME` (secret) | Token for the admin page |
| `DATABASE_URL` | `CHANGE_ME` (secret) | PostgreSQL connection string, e.g. `postgresql://vaultwarden:<password>@aac-vaultwarden-db:5432/vaultwarden` |
| `POSTGRES_DB` | `vaultwarden` | Database name for the `postgres` dependency |
| `POSTGRES_USER` | `vaultwarden` | Database user for the `postgres` dependency |
| `POSTGRES_PASSWORD` | `CHANGE_ME` (secret) | Database password |
| `DOMAIN` | `CHANGE_ME` | Public URL of the Vaultwarden instance, e.g. `https://vault.example.com` |
| `DATA_FOLDER` | `/data` | Data directory inside the container |
| `WEBSOCKET_ENABLED` | `true` | Enable WebSocket notifications for real-time sync |
| `SSO_ENABLED` | `true` | Enable OpenID Connect single sign-on |
| `SSO_AUTHORITY` | `CHANGE_ME` | OIDC issuer URL of the identity provider |
| `SSO_CLIENT_ID` | `CHANGE_ME` | OIDC client ID |
| `SSO_CLIENT_SECRET` | `CHANGE_ME` (secret) | OIDC client secret |
| `SSO_SCOPES` | `email profile offline_access` | OIDC scopes requested from the identity provider |
| `SSO_DEBUG_TOKENS` | `true` | Log SSO tokens for debugging (see note below) |
| `PUID` / `PGID` | `1000` | User and group ID the container runs as |
| `TZ` | `Europe/Berlin` | Time zone |

Ports:

| Port (container) | Host port | Protocol | Purpose |
|---|---|---|---|
| `80` | `28981` | TCP | Web vault and API (`web`) |
| `3012` | `3012` | TCP | WebSocket notifications (`websocket`) |

Volumes:

| Mount | Description |
|---|---|
| `config` -> `/data` | Vaultwarden data, attachments and configuration |
| `db` -> `/var/lib/postgresql` | PostgreSQL data (database dependency) |

Note: `SSO_DEBUG_TOKENS` is enabled in the current definition. Upstream documents it as a
debugging option; consider setting it to `false` for production use.

## Deploy with App-as-Code

1. Edit `service.yml`. It is the single source of truth for image, ports, environment,
   volumes, dependencies and per-stage overrides. Do not edit the rendered output.
2. Pushing to `main` (or a configured stage branch) runs the service pipeline, which renders
   the manifests with `aac-template-engine` and triggers the deployment.
3. Releases of this definition are published under <https://github.com/app-as-code/aac-vaultwarden>.

## Credits and license

Vaultwarden is developed by the Vaultwarden project and licensed under AGPL-3.0. This
repository only contains the deployment definition for it and is not a fork of the
application. The logo is provided by the Vaultwarden project; see
[`assets/ICON_SOURCE.md`](assets/ICON_SOURCE.md) for icon sources and licenses.
