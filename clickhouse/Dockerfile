# Telemetry store: spans, logs, metrics. The bulk of the disk and RAM.
# The official image already ships docker_related_config.xml with
# <listen_host>::</listen_host>, so it accepts Railway's IPv6-only
# private network with no extra config. 25.3 is the current LTS and
# respects cgroup memory limits — do not drop below 24.x.
FROM clickhouse/clickhouse-server:25.3

# Logs to the console instead of 10 x 1 GB files, and ClickHouse's own
# telemetry tables (trace_log, metric_log, ...) off: they write every few
# seconds and their merges keep idle memory climbing (~1 GB in 3 h).
COPY railway.xml /etc/clickhouse-server/config.d/railway.xml
# The stock file listens on :: and 0.0.0.0; on Railway :: is already
# dual-stack, so the second bind fails with an "Address already in use"
# warning on every boot. Overwritten, not deleted.
COPY listen.xml /etc/clickhouse-server/config.d/docker_related_config.xml

# Railway hosts run transparent huge pages "always": jemalloc frees 4K pages
# inside 2 MB huge pages that stay charged, so idle ClickHouse creeps ~25 MB/min.
ENV CLICKHOUSE_DB=uptrace \
    CLICKHOUSE_USER=uptrace \
    MALLOC_CONF=thp:never
