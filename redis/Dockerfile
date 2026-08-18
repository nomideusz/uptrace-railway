# go-redis RedisRing cache. Pure cache — no volume, no persistence:
# losing it costs one cold query, not data. No config file → redis binds
# all interfaces including IPv6 (required on Railway's private network).
FROM redis:7-alpine

CMD ["sh", "-c", "redis-server --save '' --appendonly no --requirepass \"$REDIS_PASSWORD\""]
