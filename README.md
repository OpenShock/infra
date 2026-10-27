# infra

Small infrastructure images and jobs for OpenShock. Each image lives in
`images/<name>/` with its own workflow in `.github/workflows/<name>.yml`
(path-filtered, so images build independently) and is published as
`ghcr.io/openshock/<name>`.

Packages inherit this repo's **private** visibility. Cluster nodes pull them
with the `ghcr-pull` secret from `kubernetes-cluster-gitops`
(`OpenShock/AppSecrets`).

## Images

### geoip

`ghcr.io/openshock/geoip`: data-only image holding the MaxMind GeoLite2 ASN
and City databases. The API and LiveControlGateway mount it as a read-only
Kubernetes image volume at `/geoip`. Rebuilt Wednesday and Saturday mornings
(UTC), just after MaxMind's Tuesday/Friday releases; pods pick up new data
when they restart.

Secrets: `MAXMIND_ACCOUNT_ID`, `MAXMIND_LICENSE_KEY`.

Must stay private: the GeoLite2 EULA forbids redistributing the databases.

### timescaledb

`ghcr.io/openshock/timescaledb:<timescaledb>-<pg major>-<debian>`: TimescaleDB
Community edition as a CloudNativePG image volume extension
([docs](https://cloudnative-pg.io/docs/1.30/imagevolume_extensions)), for the
control log hypertable and stats aggregate of the API. CloudNativePG only
publishes the Apache-2 edition, which lacks the columnstore, retention policies
and continuous aggregates. Bump `TIMESCALEDB_VERSION` in the workflow to build
a new version; after the cluster rolls onto it, run
`ALTER EXTENSION timescaledb UPDATE`.

```yaml
spec:
  imageName: ghcr.io/cloudnative-pg/postgresql:18-minimal-trixie
  postgresql:
    shared_preload_libraries: [timescaledb]
    parameters:
      timescaledb.telemetry_level: "off"
    extensions:
      - name: timescaledb
        image:
          reference: ghcr.io/openshock/timescaledb:2.31.0-18-trixie
```

Must stay private: the Timescale License forbids redistributing it.
