# Deploy and Host Plausible with Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/plausible-v3-privacy-first-analytics?referralCode=20H68q)

Plausible Community Edition is open-source, cookie-free web analytics: a lightweight script, a one-page dashboard and no personal data collection. This community template runs Plausible CE v3 on Railway with Postgres for accounts and ClickHouse for event data, tuned to stay small and inexpensive.

## About Hosting Plausible

Plausible CE needs three things: the Plausible application (an Elixir web app that serves the dashboard, the tracking script and the event API), a PostgreSQL database for users, sites and settings, and ClickHouse for the high-volume event and session data. This template starts them with pinned versions, generated secrets and a ClickHouse configuration for small machines (console logging, unused system tables removed, small caches). Plausible waits for ClickHouse, creates and migrates both databases on start, and reports health through `/api/health`. Registration is invite-only after the first account, and email can be sent through HTTP-API providers that work on every Railway plan.

## Common Use Cases

- Privacy-friendly analytics for marketing sites, blogs and documentation without cookie banners for analytics.
- Product and landing-page metrics with goals, custom events and UTM campaign tracking.
- Agencies hosting dashboards for several client sites on one instance.
- Keeping analytics data in your own infrastructure for GDPR and data-residency requirements.

## Dependencies for Plausible Hosting

- Plausible Community Edition (`ghcr.io/plausible/community-edition`)
- PostgreSQL
- ClickHouse (`clickhouse/clickhouse-server`)
- Optional: an email provider (Postmark, Mailgun, SendGrid, Mandrill, or SMTP on Railway Pro)

### Deployment Dependencies

- Plausible CE repository and compose file: https://github.com/plausible/community-edition
- Plausible CE configuration reference: https://github.com/plausible/community-edition/wiki/configuration
- Plausible documentation: https://plausible.io/docs
- ClickHouse with less than 16 GB RAM: https://clickhouse.com/docs/en/operations/tips
- Railway outbound email: https://docs.railway.com/networking/outbound-networking

### Implementation Details

| Service | Image | Role | Public |
|---|---|---|---|
| `plausible` | `ghcr.io/plausible/community-edition:v3.2.1` | Dashboard, tracking script, event API | HTTPS domain |
| `clickhouse` | `clickhouse/clickhouse-server:24.12.6.70-alpine` + low-resource config | Events and sessions (volume) | — |
| `Postgres` | Railway Postgres 18 | Users, sites, settings (volume) | — |

**First login.** When `plausible` is healthy, open its domain right away. The first account you register becomes the owner; afterwards registration is invite-only (`DISABLE_REGISTRATION=invite_only`). Add your site and paste the tracking snippet into your pages. If you attach a custom domain, update `BASE_URL`.

**Email.** For invitations, password resets and weekly reports set `MAILER_ADAPTER` and the matching API key (for example `Bamboo.PostmarkAdapter` with `POSTMARK_API_KEY`) plus `MAILER_EMAIL`. SMTP relays work only on Railway's Pro plan.

**Scaling.** Plausible and ClickHouse scale vertically: give ClickHouse more memory and CPU as traffic grows and raise `max_threads` in `services/clickhouse/users.d/low-resources-profile.xml` for faster dashboards on large sites.

**Pinning and upgrades.** Change the Plausible image tag after reading the CE release notes; migrations run on start. Keep ClickHouse on the version used by the CE `compose.yml` for that release.

### Why Deploy Plausible on Railway?

Railway runs Plausible, Postgres and ClickHouse together on a private network with persistent volumes and HTTPS out of the box. A small site costs a few dollars a month in usage, and you keep full ownership of your analytics data.
