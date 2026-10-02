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
