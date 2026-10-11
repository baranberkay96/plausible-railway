# NOTICE — Plausible template

This is a community template. It is not affiliated with or endorsed by Plausible Analytics or ClickHouse Inc.

## Adapted files (MIT)

### railwayapp-templates/plausible

`services/clickhouse/config.d/railway-logs.xml`, `services/clickhouse/config.d/low-resources.xml` and the Dockerfile structure adapt `packages/clickhouse/ch-config.xml`, `ch-user-config.xml` and `dockerfile.clickhouse` from https://github.com/railwayapp-templates/plausible (template `mzYEXO`). Changes: pinned ClickHouse version, no credentials baked into the image, settings merged with current Plausible CE upstream files, `max_concurrent_queries` / `max_execution_time` limits not carried over.

```
MIT License

Copyright (c) 2023 Railway

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### plausible/community-edition

`services/clickhouse/config.d/railway-logs.xml` (query_log TTL definition, removed system logs), `config.d/low-resources.xml` (`mark_cache_size`) and `users.d/low-resources-profile.xml` (profile settings) adapt `clickhouse/logs.xml`, `clickhouse/low-resources.xml` and `clickhouse/default-profile-low-resources-overrides.xml` at tag `v3.2.1` of https://github.com/plausible/community-edition. Start command and image tags follow its `compose.yml`.

```
MIT License

Copyright (c) 2024 Plausible Analytics

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## Upstream sources read (not copied)

| Source | Version | License | Use |
|---|---|---|---|
| plausible/analytics `config/runtime.exs`, `Dockerfile`, `lib/plausible_web/plugs/first_launch_plug.ex` | `v3.2.1` | AGPL-3.0 | Env-var names and defaults, `ECTO_IPV6` deprecation, first-launch registration behaviour |
| Plausible CE wiki "Configuration" | 2026-10 | — | Variable documentation |
| ClickHouse `docker/server/entrypoint.sh` | 24.12 | Apache-2.0 | `CLICKHOUSE_USER/PASSWORD/DB` handling |
| Railway docs "Outbound Networking" | 2026-10 | — | SMTP only on Pro |

## Market templates reviewed, not used

`plausible-community-edition` → github.com/gridalpha/plausible-clickhouse-railway (no license, nothing copied); `plausible` (image-based, no repo).

## Runtime images

`ghcr.io/plausible/community-edition:v3.2.1` (Plausible, AGPL-3.0, unmodified), `clickhouse/clickhouse-server:24.12.6.70-alpine` (Apache-2.0, config files added), `ghcr.io/railwayapp-templates/postgres-ssl:18` (copied from `_railway-base/postgres.json`).
