# Plausible v3 Privacy-First Analytics — composer spec

| Field | Value |
|---|---|
| Listing name | **Plausible v3 Privacy-First Analytics** (version label `v3` = Plausible CE major; the project tags `v3.2.1`) |
| Description | Cookie-free, lightweight web analytics you own, powered by ClickHouse. (77 chars, unchanged) |
| Category | Analytics |
| Tags | plausible, web-analytics, privacy, gdpr, google-analytics-alternative, clickhouse |
| Base market template | `mzYEXO` (Railway's own "Plausible Analytics", 929 projects; source `railwayapp-templates/plausible`, **MIT**) — ClickHouse low-resource config adapted with attribution. Also reviewed `plausible` (27 projects) and `plausible-community-edition` (gridalpha ClickHouse repo, no license → not used). |
| Upstream checked | Plausible CE `v3.2.1` (2026-05-15): `plausible/community-edition` `compose.yml` + `clickhouse/*.xml` (MIT); `plausible/analytics` `v3.2.1` `config/runtime.exs`, `Dockerfile`, `lib/plausible_web/plugs/first_launch_plug.ex` (AGPL-3.0, read only); CE wiki "Configuration". Pinned 2026-10-10. |

## Architecture

```
 visitors' browsers ──script.js / /api/event──► plausible :8000 (public domain)
 you ──dashboard──────────────────────────────► plausible
                                                   │            │
                         Postgres (users, sites) ◄─┘            └─► clickhouse :8123 (events, sessions)
```

## Services

| Service | Source | Start command | Public | Healthcheck | Volume | Restart |
|---|---|---|---|---|---|---|
| `plausible` | image `ghcr.io/plausible/community-edition:v3.2.1` | `/bin/sh -c '<wait for clickhouse /ping>; /entrypoint.sh db createdb && /entrypoint.sh db migrate && exec /entrypoint.sh run'` | domain → 8000 | `/api/health`, 600 s | — (stateless) | ON_FAILURE ×10 |
| `clickhouse` | repo `/services/clickhouse` (FROM `clickhouse/clickhouse-server:24.12.6.70-alpine`) | image entrypoint | — | `/ping` on PORT=8123, 300 s | `/var/lib/clickhouse` | ON_FAILURE ×10 |
| `Postgres` | `ghcr.io/railwayapp-templates/postgres-ssl:18` (from `_railway-base`) | image | — | — | `/var/lib/postgresql/data` | ON_FAILURE ×10 |

The start command is upstream's (`db createdb && db migrate && run`), preceded by a bounded wait (≤ 5 min) for ClickHouse because Railway starts all services at once and the ClickHouse image is still building when Plausible starts.

## Variables

### plausible
| Variable | Default | Notes |
|---|---|---|
| `PORT` | `8000` | |
| `BASE_URL` | `https://${{RAILWAY_PUBLIC_DOMAIN}}` | Update when adding a custom domain (tracking snippet and links use it). |
| `SECRET_KEY_BASE` | `${{secret(86, alnum)}}` | ≥ 64 bytes as the CE wiki requires. |
| `TOTP_VAULT_KEY` | `${{secret(43, base64)}}=` | 32 bytes, Base64; encrypts 2FA secrets. |
| `DATABASE_URL` | `${{Postgres.DATABASE_URL}}` | |
| `CLICKHOUSE_DATABASE_URL` | `http://${{clickhouse.CLICKHOUSE_USER}}:${{clickhouse.CLICKHOUSE_PASSWORD}}@${{clickhouse.RAILWAY_PRIVATE_DOMAIN}}:8123/${{clickhouse.CLICKHOUSE_DB}}` | |
| `CLICKHOUSE_HOST` | `${{clickhouse.RAILWAY_PRIVATE_DOMAIN}}` | Wait loop only. |
| `DISABLE_REGISTRATION` | `invite_only` | See "Registration". |
| `ENABLE_EMAIL_VERIFICATION` | `false` | Enable only with working email. |

Optional email (pick one):
- **HTTP API providers (work on every Railway plan):** `MAILER_ADAPTER=Bamboo.PostmarkAdapter` + `POSTMARK_API_KEY`, or `Bamboo.MailgunAdapter` + `MAILGUN_API_KEY`, `MAILGUN_DOMAIN` (+ `MAILGUN_BASE_URI` for EU), or `Bamboo.SendGridAdapter` + `SENDGRID_API_KEY`, or `Bamboo.MandrillAdapter` + `MANDRILL_API_KEY`. Set `MAILER_EMAIL` (verified sender) and optionally `MAILER_NAME`.
- **SMTP relay (Railway Pro plan only — outbound SMTP is disabled on Trial/Hobby):** keep the default `Bamboo.Mua` adapter and set `SMTP_HOST_ADDR`, `SMTP_HOST_PORT` (587), `SMTP_USER_NAME`, `SMTP_USER_PWD`, `SMTP_HOST_SSL_ENABLED`.

Optional: `MAXMIND_LICENSE_KEY` (+ `MAXMIND_EDITION`, default `GeoLite2-City`) for city-level geolocation; without it the bundled DB-IP country database is used. `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` for Search Console and Google Analytics imports.

### clickhouse
| Variable | Default | Notes |
|---|---|---|
| `PORT` | `8123` | |
| `CLICKHOUSE_DB` | `plausible_events_db` | Created by the official entrypoint. |
| `CLICKHOUSE_USER` | `plausible` | Entrypoint removes the passwordless `default` user and creates this one (`::/0`). |
| `CLICKHOUSE_PASSWORD` | `${{secret(32, alnum)}}` | |
| `CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT` | `0` | |
| `RAILWAY_DEPLOYMENT_DRAINING_SECONDS` | `60` | |

## Registration

Plausible CE has no admin bootstrap variable. On first launch every page redirects to the registration form and **the first person to register becomes the owner**. After that, `DISABLE_REGISTRATION=invite_only` (CE default) means only people you invite can create accounts. Recommended: open the generated domain and register immediately after the first deploy (the random `*.up.railway.app` domain is not published anywhere). Set `DISABLE_REGISTRATION=true` afterwards to block even invited sign-ups. Inviting team members by email needs one of the email options above.

## Composer steps (Railway dashboard)

1. Add `Postgres` from `_railway-base/postgres.json`.
2. Create `clickhouse` from GitHub repo `baranberkay96/plausible-railway`, root `/services/clickhouse`; volume `/var/lib/clickhouse`; variables; healthcheck `/ping`.
3. Create `plausible` from image `ghcr.io/plausible/community-edition:v3.2.1`; start command as above; variables; domain → 8000; healthcheck `/api/health`, timeout 600.

## Improvements over market templates

| Area | Market (`mzYEXO` and others) | This template |
|---|---|---|
| Versions | `community-edition:v2` / `:latest`, ClickHouse `latest-alpine`, Postgres `postgres-ssl:latest` | Plausible CE `v3.2.1`, ClickHouse `24.12.6.70-alpine` (the version CE's compose uses), Postgres 18 base |
| Secrets | `mzYEXO` bakes ClickHouse credentials into the image via build ARG→ENV | Runtime-only credentials via the official entrypoint |
| ClickHouse user | `default` user with password (mzYEXO) | Dedicated `plausible` user, passwordless `default` removed, SQL access management off |
| Config | Railway low-resource config (2023) | Merged with current CE upstream low-resource + logging files: console-only logs, 30-day `query_log`, unused system tables removed, small mark cache, single-thread profile |
| Deprecated vars | `ECTO_IPV6`, `ECTO_CH_IPV6`, `ENABLE_ALPINE_PRIVATE_NETWORKING`, `DATABASE_PRIVATE_URL` | Removed (CE v3 tries IPv6 then IPv4 automatically) |
| Start-up | `nc` loop without limit or no wait at all | Bounded wait on ClickHouse `/ping`, then upstream's createdb/migrate/run |
| Health | `/` or none | `/api/health` (checks Postgres + ClickHouse) |
| Email | SMTP variables only (blocked on Hobby) | HTTP-API mailers documented for Hobby, SMTP for Pro |
| Registration | undocumented | First-owner behaviour and invite-only explained |

## Resource profile and cost (Railway: RAM $10/GB-mo, vCPU $20/mo, volume $0.15/GB-mo)

| Service | Idle RAM | Typical RAM | Idle CPU | Typical CPU | Volume |
|---|---|---|---|---|---|
| plausible | 0.25 GB | 0.35–0.5 GB | 0.02 | 0.05–0.2 | — |
| clickhouse (low-resource profile) | 0.3 GB | 0.5–1.0 GB | 0.02 | 0.05–0.3 | 1–5 GB |
| Postgres | 0.08 GB | 0.12 GB | 0.01 | 0.02 | < 1 GB |
| **Total** | **~0.65 GB** | **~1–1.6 GB** | **~0.05 vCPU** | **~0.1–0.5 vCPU** | **~2–6 GB** |

Estimated monthly cost: **idle ≈ $8**; **typical (up to ~1M pageviews/month) ≈ $12–25**; larger sites mostly add ClickHouse RAM/CPU and volume. Minimum plan: **Hobby** (Trial works for evaluation but its credit runs out). Pro needed only for SMTP email or very large traffic.

## Pinning and upgrades

Pinned 2026-10-10: `ghcr.io/plausible/community-edition:v3.2.1`, `clickhouse/clickhouse-server:24.12.6.70-alpine`, `ghcr.io/railwayapp-templates/postgres-ssl:18`. Upgrade Plausible by changing the image tag after reading the CE release notes (migrations run on start). Change the ClickHouse `FROM` only to a version the CE `compose.yml` uses.

## Verification checklist

- [ ] `clickhouse` healthy on `/ping`; logs show user `plausible` and database `plausible_events_db` created on first boot.
- [ ] `plausible` logs: waits for ClickHouse, `createdb`, `migrate` (Postgres + ClickHouse), then listens on 8000; `/api/health` returns 200.
- [ ] Open the domain → registration page → create the owner account → add a site.
- [ ] Add the snippet to a test page (or `curl -X POST https://<domain>/api/event -H 'Content-Type: application/json' -H 'User-Agent: Mozilla/5.0' -d '{"name":"pageview","url":"https://example.com/","domain":"<site>"}'` → 202) and see the visit in the real-time view.
- [ ] Log out, try `/register` → blocked (invite-only).
- [ ] Redeploy all services: account, sites and stats persist.

## Open questions

- Not deployed end-to-end here. ClickHouse 24.12 is what Plausible CE v3.2.1 ships with but is past ClickHouse's own support window; follow CE releases for the next supported version rather than upgrading ClickHouse independently.
- Postgres 18 (Railway base) instead of upstream's 16: Plausible's Ecto migrations are standard SQL and are expected to work, but CE's compose is tested with 16.
- No volume on `plausible`: upstream mounts `/var/lib/plausible` for large CSV import temp files and MaxMind downloads. If you import big Google Analytics exports or enable MaxMind, add a volume there (and `RAILWAY_RUN_UID=0`, since the image runs as UID 999).
