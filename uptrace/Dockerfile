# Uptrace itself: OTLP ingest + the UI. The upstream entrypoint runs
# `/uptrace --config=/etc/uptrace/config.yml serve`, and the image ships
# no config of its own — dropping ours in is the whole wrapper.
FROM uptrace/uptrace:2.0.3

COPY config.yml /etc/uptrace/config.yml
