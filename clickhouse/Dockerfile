# Telemetry store: spans, logs, metrics. The bulk of the disk and RAM.
# The official image already ships docker_related_config.xml with
# <listen_host>::</listen_host>, so it accepts Railway's IPv6-only
# private network with no extra config. 25.3 is the current LTS and
# respects cgroup memory limits — do not drop below 24.x.
FROM clickhouse/clickhouse-server:25.3

ENV CLICKHOUSE_DB=uptrace \
    CLICKHOUSE_USER=uptrace
