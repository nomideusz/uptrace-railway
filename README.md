# Deploy and Host Uptrace on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/uptrace?utm_medium=integration&utm_source=button&utm_campaign=uptrace)

[Uptrace](https://uptrace.dev/) is an open-source APM built on OpenTelemetry and ClickHouse. It ingests traces, logs, and metrics through the standard OTLP protocol, stores them in a column store built for the job, and gives you distributed tracing, a service graph, dashboards, and alerting behind one UI — no per-seat pricing and no data leaving your Railway project.

**Read this before deploying: OTLP/HTTP is public, OTLP/gRPC is not.** Railway routes one public port per service, and here it serves both the UI and OTLP/HTTP ingest. That covers every OpenTelemetry SDK and the Collector's `otlphttp` exporter over your Railway domain. gRPC ingest listens on `4317` and is reachable from other services in the same project over the private network — from outside, it needs a TCP proxy on that port. If your telemetry comes from outside Railway, use OTLP/HTTP.

## About Hosting Uptrace

Uptrace splits its state across three stores, and this template runs each as its own service so they can be sized and scaled apart. ClickHouse holds the telemetry — spans, logs, and metrics — and is the service that will actually grow; it gets a volume and it is where your storage bill lives. Postgres holds metadata: users, projects, dashboards, and alert rules, and stays small. Redis is a pure cache with no volume at all, because losing it costs one cold query rather than any data. The Uptrace service itself is stateless.

The whole configuration is one `config.yml` baked into a thin wrapper image, and every value that varies per deploy is an environment variable — Uptrace expands `${VAR}` in that file itself, so there is no entrypoint script and nothing is generated at build time. Wiring between the four services is done with Railway reference variables, so passwords are generated once and never typed.

## Common Use Cases

- A self-hosted APM for the apps already running in the same Railway project, sending OTLP over the private network with no egress.
- A Datadog or New Relic replacement for a small team, where per-host or per-seat pricing costs more than the infrastructure does.
- A trace store you can query directly — the data sits in your own ClickHouse, so anything ClickHouse SQL can answer, you can answer.
- A staging observability stack that mirrors production tooling without a second vendor contract.

## Dependencies for Uptrace Hosting

- A ClickHouse service with a volume (created by this template) for telemetry.
- A Postgres service with a volume (created by this template) for metadata.
- A Redis service (created by this template) as cache.
- A Railway domain on the `uptrace` service for the UI and OTLP/HTTP ingest.

### Deployment Dependencies

- [Uptrace documentation](https://uptrace.dev/get)
- [Uptrace source](https://github.com/uptrace/uptrace)
- [Template source on GitHub](https://github.com/nomideusz/uptrace-railway)
- [OpenTelemetry OTLP exporter docs](https://opentelemetry.io/docs/specs/otlp/)

### Implementation Details

**Four services from one repo: `uptrace/` wraps the official `uptrace/uptrace:2.0.3` image with a config file, and `clickhouse/`, `postgres/`, and `redis/` are three-line Dockerfiles over the official images.**

1. Deploy, then open the generated domain. Sign in with `UPTRACE_ADMIN_EMAIL` and the generated `UPTRACE_ADMIN_PASSWORD`, both on the `uptrace` service. Change the password in the UI afterwards — the seeded account is created on first boot only, and editing the variable later does nothing.
2. Point your app at the DSN `https://<UPTRACE_PROJECT_TOKEN>@<your-railway-domain>`, taking the token from the `uptrace` service's variables. It is seeded into the config, so it works before you ever log in. For apps inside the same Railway project, use `http://<token>@${{uptrace.RAILWAY_PRIVATE_DOMAIN}}:8080` and keep the traffic off the public internet.
3. Any OpenTelemetry SDK works without an Uptrace-specific library: set `OTEL_EXPORTER_OTLP_ENDPOINT` to `https://<your-railway-domain>`, `OTEL_EXPORTER_OTLP_PROTOCOL` to `http/protobuf`, and `OTEL_EXPORTER_OTLP_HEADERS` to `uptrace-dsn=<your DSN>`.
4. Alert emails and password resets need SMTP. Set `UPTRACE_SMTP_ENABLED=true` on the `uptrace` service along with `UPTRACE_SMTP_HOST`, `UPTRACE_SMTP_PORT`, `UPTRACE_SMTP_USERNAME`, `UPTRACE_SMTP_PASSWORD`, and `UPTRACE_SMTP_FROM`.

**Things worth knowing before you deploy:**

- **ClickHouse decides your bill.** Telemetry is high-volume by nature and this template ships with no retention limit, so the volume grows until you set one. Use sampling in your SDK and set retention per project in the UI before you point production at it.
- **Self-monitoring is off.** Uptrace can trace itself, but an *idle* instance emits roughly 900 spans a minute of its own Redis and Postgres chatter — about 1.3M spans a day of storage you pay for, in the same project as your app's data. Set `UPTRACE_SELF_MONITORING_DISABLED=false` if you want it anyway.
- **Give ClickHouse room.** It is the memory-hungry service here; if queries get killed or ingest stalls, that is the service to scale, not `uptrace`.
- **The first boot runs migrations** against both databases before the healthcheck can pass, which is why `RAILWAY_HEALTHCHECK_TIMEOUT_SEC` is set to 600. Leave it.
- **Only the `uptrace` service has a healthcheck, by design.** Railway runs healthchecks against services with a public domain and requires a 2xx from the routed port; on internal-only services like ClickHouse, Postgres, and Redis the setting is accepted and never executed, so adding one there buys nothing. Note also that Uptrace serves its SPA for any unmatched path, so the healthcheck proves the HTTP server is up but not that ClickHouse is reachable — watch the logs, not the green check, when ingest looks wrong.

## Why Deploy Uptrace on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying Uptrace on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
