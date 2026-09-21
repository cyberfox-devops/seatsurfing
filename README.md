# Seatsurfing

[![](https://img.shields.io/github/v/release/seatsurfing/seatsurfing)](https://github.com/seatsurfing/seatsurfing/releases)
[![](https://img.shields.io/github/release-date/seatsurfing/seatsurfing)](https://github.com/seatsurfing/seatsurfing/releases)
[![](https://img.shields.io/github/actions/workflow/status/seatsurfing/seatsurfing/release.yml?branch=main)](https://github.com/seatsurfing/seatsurfing/actions)
[![](https://img.shields.io/github/license/seatsurfing/seatsurfing)](https://github.com/seatsurfing/seatsurfing/blob/main/LICENSE)

## 🚀 Seatsurfing SaaS available!

We offer [Seatsurfing](https://seatsurfing.io/) as a fully-hosted Software-as-a-Service (SaaS). [Start for free now](https://seatsurfing.io/sign-up)!

- **No installation required** - Get started immediately
- **Microsoft Teams integration** - See [Microsoft AppSource marketplace](https://appsource.microsoft.com/product/office/WA200008773)
- **Get it free** - Free for up to 10 users
- **Automatic updates** - Always enjoy the latest features
- **Managed infrastructure** - Servers in Germany (EU)

## 📖 Introduction

Seatsurfing is a software which enables your organization's employees to book seats, desks and rooms.

This repository contains the Backend, which consists of:

- The Server (REST API Backend) written in Go
- User Self-Service Booking Web Interface ("Booking UI"), built as a Progressive Web Application (PWA) which can be installed on mobile devices
- Admin Web Interface ("Admin UI")
- Common TypeScript files for the two TypeScript/React web frontends

**[Visit project's website for more information.](https://seatsurfing.io)**

## 📷 Screenshots

### Web Admin UI

![Seatsurfing Web Admin UI](https://raw.githubusercontent.com/seatsurfing/seatsurfing/main/.github/admin-ui.png)

### Web Booking UI

![Seatsurfing Web Booking UI](https://raw.githubusercontent.com/seatsurfing/seatsurfing/main/.github/booking-ui.png)

## 🗸 Quick reference

- **Maintained by:** [seatsurfing.io](https://seatsurfing.io/)
- **Where to get help:** [Documentation](https://seatsurfing.io/docs/)
- **Docker architectures:** [amd64, arm64](https://github.com/seatsurfing/seatsurfing/pkgs/container/backend)
- **License:** [GPL 3.0](https://github.com/seatsurfing/seatsurfing/blob/main/LICENSE)

## 🐋 How to use the Docker image

### Start using Docker Compose

```
services:
  server:
    image: ghcr.io/seatsurfing/backend
    restart: always
    networks:
      sql:
    ports:
      - 8080:8080
    environment:
      POSTGRES_URL: 'postgres://seatsurfing:DB_PASSWORD@db/seatsurfing?sslmode=disable'
      # CRYPT_KEY is used for encrypting sensitive data in the database
      CRYPT_KEY: 'some-random-32-bytes-long-string'
      # When running without a reverse proxy taking care of TLS termination,
      # use the following settings which enable HTTP on port 8080
      PUBLIC_SCHEME: 'http'
      PUBLIC_PORT: '8080'
  db:
    image: postgres:17
    restart: always
    networks:
      sql:
    volumes:
      - db:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: DB_PASSWORD
      POSTGRES_USER: seatsurfing
      POSTGRES_DB: seatsurfing

volumes:
  db:

networks:
  sql:
```

This starts …

- … a PostgreSQL database with data stored on Docker volume "db"
- … a Seatsurfing instance with port 8080 exposed

The Seatsurfing Booking UI is accessible at :8080/ui/search/ and the Seatsurfing Admin UI instance at :8080/ui/admin/.

To login, use the default admin login (user `admin@seatsurfing.local` and password `Sea!surf1ng`) or set the [environment variables](https://seatsurfing.io/docs/self-hosted/config) `INIT_ORG_USER` and `INIT_ORG_PASS` to customize the admin login.

### Running on Kubernetes

Please refer to our [Kubernetes documentation](https://seatsurfing.io/docs/self-hosted/kubernetes/).

## ⚙️ Environment variables

Please check out the [documentation](https://seatsurfing.io/docs/self-hosted/config) for information on available environment variables and further guidance.

**Hint**: When running in an IPV6-only Docker/Podman environment with multiple network interfaces bound to the Frontend containers, setting the `LISTEN_ADDR` environment variable can be necessary as NextJS binds to only one network interface by default. Set it to `::` to bind to any address.

---

## Operations runbook (meet.cyberfox.com)

_Last verified 2026-09-21._

### Where it runs
| Item | Value |
|---|---|
| Public hostname | `meet.cyberfox.com` (alias `seatsurfing.cyberfox.com`) → `40.114.34.159` |
| Azure | RG `cyberfox-seatsurfing-rg` (East US), VM `seatsurfing-vm` (Ubuntu, Docker Compose), ACR `cyberfoxseatsurfing` |
| Compose project | `/home/azureuser/` — `docker-compose.yml`, `docker-compose.override.yml` (SMTP), `.env` (secrets, chmod 600) |
| Containers | `azureuser-server-1` (`cyberfoxseatsurfing.azurecr.io/seatsurfing-backend:branded`, built from branch `cyberfox-branding`), `azureuser-db-1` (`postgres:17`, volume `db`) |
| Backend port | `127.0.0.1:8080` on the VM, fronted by the TLS proxy on 443 |
| Admin UI | `https://meet.cyberfox.com/ui/admin/` — local login form at `/ui/login` |
| Mail relay | `ss-mail-relay.service` on the VM (`/opt/ss-mail-relay`, env `/etc/ss-mail-relay/relay.env`), listens `0.0.0.0:2525`, sends via Graph as `donotreply@cyberfox.com`. Port 2525 must never be opened on `seatsurfing-vmNSG`. |

### Secrets — never leave these single-copy again
All in Key Vault `threatiq-kv-prod`; the VM's system-assigned managed identity has **Key Vault Secrets User** on the vault.

| Secret name | Used by |
|---|---|
| `seatsurfing-crypt-key` | `CRYPT_KEY` in `.env` — encrypts data at rest (SSO client secret). Rotated 2026-09-21 after the original was lost; the SSO client secret was re-entered afterward. |
| `seatsurfing-db-password` | `DB_PASSWORD` in `.env` — Postgres password for user `seatsurfing` |
| `seatsurfing-mailer-client-secret` | Mail relay — client secret on the shared `CyberFOX-NoReply-Mailer` app registration (`efb68e07-da8e-4aa5-ad37-7bef1f0da150`), consumer tag `seatsurfing-mailer` |

Entra app registrations: **Seatsurfing SSO** `8eb0814f-af8c-4b4c-babf-342af960a69f` (login, redirect on meet.cyberfox.com); **Seatsurfing M365 Sync Worker** `d004750b-7580-4eac-903c-399c3aeb89f7` (`Calendars.ReadWrite`, `Place.Read.All` — writes bookings to the room mailboxes).

**Incident 2026-09-21:** `DB_PASSWORD` / `CRYPT_KEY` had only ever existed as shell exports in the July 17 install session. A `docker compose up -d server` run from a fresh shell recreated the backend with blank values and took the site down. Fix: `.env` in the project directory (compose reads it automatically) plus the vault copies above. **Never run compose without `.env` present.**

### Running commands on the VM (no SSH key needed)
Azure run-command executes as root. Output is capped at 4,096 characters. Write the script to a file first — PowerShell splits inline scripts on embedded double quotes.
```powershell
'sudo docker ps' | Set-Content -Path "$HOME\ss.sh" -NoNewline -Encoding ascii
az vm run-command invoke --resource-group cyberfox-seatsurfing-rg --name seatsurfing-vm --command-id RunShellScript --scripts "@$HOME\ss.sh" --query "value[0].message" -o tsv
```
Useful one-liners for the script file:
- Backend log: `docker logs --tail 80 azureuser-server-1 2>&1 | grep -iv "smtp\|reminder\|healthcheck"`
- Relay log: `journalctl -u ss-mail-relay -n 30 --no-pager`
- Restart backend safely: `cd /home/azureuser && docker compose up -d server`
- SQL: `docker exec azureuser-db-1 psql -U seatsurfing -d seatsurfing -c "SELECT email, role FROM users ORDER BY role DESC;"`

### Users and roles
Anyone on `cyberfox.com`, `passwordboss.com`, or `connecton.com` who clicks **Sign in with Microsoft** is auto-created as a **User**. Invite emails are only needed for password logins; SSO users can just sign in.

Roles (DB `users.role`): `0` User · `10` Space Admin · `20` Org Admin · `22` service account (`sync-worker@seatsurfing.local` — leave alone) · `90` Super Admin.

- Change a role: **Users → person → Role → Save**.
- Offboard: **Users → person → Delete**.
- Add a domain: **Settings → Organization** (a domain must exist before users on it can be created).
- Emergency promote via SQL: `UPDATE users SET role = 20 WHERE email = 'someone@cyberfox.com';`
- Break-glass local admin: `admin@seatsurfing.local` (Org Admin, password login at `/ui/login`).

**Symptom guide:** a "Server error" popup on Dashboard with `403` on `/stats/` in DevTools = the signed-in account is not an Org Admin. `500` on `PUT /user/…` when sending an invite = mail relay down (`systemctl status ss-mail-relay`).

### Known gaps
- Backend is 1.115.0; upstream is 1.127.9. Rebuild `branded` from `cyberfox-branding` after rebasing.
- `JWT_PRIVATE_KEY` is not set, so every restart logs all users out. Generate a key pair, store in the vault, add to `.env`.
- Booking-reminder emails were silently failing from 2026-07-17 to 2026-09-21 (no SMTP configured).
