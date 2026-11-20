# infra

Small infrastructure images and jobs for OpenShock. Each image lives in
`images/<name>/` with its own workflow in `.github/workflows/<name>.yml`
(path-filtered, so images build independently) and is published as
`ghcr.io/openshock/<name>`.

Package visibility is set per image (see below). Cluster nodes pull private
ones with the `ghcr-pull` secret from `kubernetes-cluster-gitops`
(`OpenShock/AppSecrets`).

## License

The contents of this repo are [MIT](LICENSE). That covers the Dockerfiles and
workflows only, not the third-party software and data baked into the images,
which keep their own licenses.

## Images

### geoip

`ghcr.io/openshock/geoip`: data-only image holding the MaxMind GeoLite2 ASN
and City databases. The API and LiveControlGateway mount it as a read-only
Kubernetes image volume at `/geoip`. Rebuilt Wednesday and Saturday mornings
(UTC), just after MaxMind's Tuesday/Friday releases; pods pick up new data
when they restart.

Secrets: `MAXMIND_ACCOUNT_ID`, `MAXMIND_LICENSE_KEY`.

**Private.** The GeoLite2 EULA forbids redistributing the databases.

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
          reference: ghcr.io/openshock/timescaledb:2.32.1-18-trixie
```

**Public.** The image carries the TimescaleDB core (Apache-2.0) and the `tsl`
module under the
[Timescale License](https://github.com/timescale/timescaledb/blob/main/tsl/LICENSE-TIMESCALE),
declared in its `org.opencontainers.image.licenses` label as
`Apache-2.0 AND LicenseRef-Timescale`. The Timescale License (2.1(c)) allows
redistributing the binaries in unmodified standalone form, so this image only
copies them out of Timescale's official apt packages; don't patch or rebuild
them. It forbids offering TimescaleDB as a database-as-a-service or
distributing it for that use.

### postgres-timescaledb

`ghcr.io/openshock/postgres-timescaledb:<timescaledb>-<pg major>.<minor>-<debian>`:
the official `postgres` image with the TimescaleDB Community edition installed
and preloaded, used by the API repo's `docker-compose.yml` (self-hosting) and
`Dev/docker-compose.yml`. It keeps the official image's Debian base, so data
directories created by plain `postgres:18` keep their collations and can be
reused.

Tags are immutable. A daily run checks `postgres:<pg major>-<debian>` and
publishes a new tag when it has a Postgres minor release that isn't built yet;
bump the tag in both compose files to roll it out. New TimescaleDB versions
are manual: bump `TIMESCALEDB_VERSION` in the workflow and the compose tags,
then run `ALTER EXTENSION timescaledb UPDATE`.

**Public.** Self-hosters pull it. Same licensing as `timescaledb` above: the
binaries come unmodified from Timescale's apt packages.
