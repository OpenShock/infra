# postgres with the TimescaleDB Community edition, for docker compose (self-hosting and development). Built on the
# official Debian image so existing data directories keep their glibc/ICU collations; BASE pins the Debian release.
ARG BASE=postgres:18-trixie
FROM $BASE

ARG TIMESCALEDB_VERSION
ARG BASE_DIGEST

ADD https://packagecloud.io/timescale/timescaledb/gpgkey /etc/apt/keyrings/timescaledb.asc

RUN set -eux; \
    test -n "$TIMESCALEDB_VERSION"; \
    chmod 644 /etc/apt/keyrings/timescaledb.asc; \
    . /etc/os-release; \
    apt-get update; \
    apt-get install -y --no-install-recommends ca-certificates; \
    echo "deb [signed-by=/etc/apt/keyrings/timescaledb.asc] https://packagecloud.io/timescale/timescaledb/debian/ $VERSION_CODENAME main" \
        > /etc/apt/sources.list.d/timescaledb.list; \
    apt-get update; \
    version="$(apt-cache madison "timescaledb-2-postgresql-$PG_MAJOR" | awk '{ print $3 }' | grep -m1 "^$TIMESCALEDB_VERSION~")"; \
    apt-get install -y --no-install-recommends \
        "timescaledb-2-postgresql-$PG_MAJOR=$version" \
        "timescaledb-2-loader-postgresql-$PG_MAJOR=$version"; \
    test -f "/usr/lib/postgresql/$PG_MAJOR/lib/timescaledb-tsl-$TIMESCALEDB_VERSION.so"; \
    rm -rf /var/lib/apt/lists/*

LABEL org.opencontainers.image.source="https://github.com/OpenShock/infra"
LABEL org.opencontainers.image.description="PostgreSQL with the TimescaleDB Community edition preloaded"
# Core is Apache-2.0, the tsl module is under the Timescale License (no SPDX id):
# https://github.com/timescale/timescaledb/blob/main/tsl/LICENSE-TIMESCALE
LABEL org.opencontainers.image.licenses="PostgreSQL AND Apache-2.0 AND LicenseRef-Timescale"
# The base this tag was built from
LABEL org.opencontainers.image.base.digest="$BASE_DIGEST"

CMD ["postgres", "-c", "shared_preload_libraries=timescaledb", "-c", "timescaledb.telemetry_level=off"]
