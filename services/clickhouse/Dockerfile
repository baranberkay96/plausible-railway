# ClickHouse for Plausible Community Edition on Railway, tuned for small instances.
# Version: the ClickHouse release Plausible CE v3.2.1's official compose.yml uses.
#
# Adapted from github.com/railwayapp-templates/plausible (packages/clickhouse),
# Copyright (c) 2023 Railway, MIT License, and from
# github.com/plausible/community-edition (clickhouse/*.xml),
# Copyright (c) 2024 Plausible Analytics, MIT License. See NOTICE.md.
#
# Unlike the original, no credentials are baked into the image: the official
# entrypoint reads CLICKHOUSE_USER / CLICKHOUSE_PASSWORD / CLICKHOUSE_DB at runtime.
FROM clickhouse/clickhouse-server:24.12.6.70-alpine

COPY config.d/ /etc/clickhouse-server/config.d/
COPY users.d/ /etc/clickhouse-server/users.d/

EXPOSE 8123 9000
