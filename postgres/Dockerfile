# Metadata: users, projects, dashboards, alert rules. Small and boring.
# PGDATA one level below the mount: Railway volumes arrive with lost+found,
# which initdb refuses to initialize into.
FROM postgres:17-alpine

ENV POSTGRES_USER=uptrace \
    POSTGRES_DB=uptrace \
    PGDATA=/var/lib/postgresql/data/pgdata
