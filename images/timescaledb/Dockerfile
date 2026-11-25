# CloudNativePG image volume extension with the TimescaleDB Community edition, CloudNativePG only publishes the
# Apache-2 edition. BASE must match the cluster's imageName in PostgreSQL major and Debian release.
ARG BASE=ghcr.io/cloudnative-pg/postgresql:19-minimal-trixie
FROM $BASE AS builder

ARG PG_MAJOR=18
ARG TIMESCALEDB_VERSION

USER 0

ADD https://packagecloud.io/timescale/timescaledb/gpgkey /etc/apt/keyrings/timescaledb.asc

RUN set -eux;     test -n "$TIMESCALEDB_VERSION";     chmod 644 /etc/apt/keyrings/timescaledb.asc;     . /etc/os-release;     apt-get update;     apt-get install -y --no-install-recommends ca-certificates;     echo "deb [signed-by=/etc/apt/keyrings/timescaledb.asc] https://packagecloud.io/timescale/timescaledb/debian/ $VERSION_CODENAME main"         > /etc/apt/sources.list.d/timescaledb.list;     apt-get update;     version="$(apt-cache madison "timescaledb-2-postgresql-$PG_MAJOR" | awk '{ print $3 }' | grep -m1 "^$TIMESCALEDB_VERSION~")";     apt-get install -y --no-install-recommends         "timescaledb-2-postgresql-$PG_MAJOR=$version"         "timescaledb-2-loader-postgresql-$PG_MAJOR=$version";     test -f "/usr/lib/postgresql/$PG_MAJOR/lib/timescaledb-tsl-$TIMESCALEDB_VERSION.so"

FROM scratch
ARG PG_MAJOR=18
LABEL org.opencontainers.image.source="https://github.com/OpenShock/infra"
LABEL org.opencontainers.image.description="TimescaleDB Community edition as a CloudNativePG image volume extension"
# Core is Apache-2.0, the tsl module is under the Timescale License (no SPDX id):
# https://github.com/timescale/timescaledb/blob/main/tsl/LICENSE-TIMESCALE
LABEL org.opencontainers.image.licenses="Apache-2.0 AND LicenseRef-Timescale"

COPY --from=builder /usr/lib/postgresql/${PG_MAJOR}/lib/timescaledb* /lib/
COPY --from=builder /usr/share/postgresql/${PG_MAJOR}/extension/timescaledb* /share/extension/

USER 65532:65532
