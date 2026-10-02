# Data-only image: mounted into the API and Gateway as a read-only Kubernetes image volume.
FROM scratch
LABEL org.opencontainers.image.source="https://github.com/OpenShock/infra"
LABEL org.opencontainers.image.description="MaxMind GeoLite2 ASN + City databases"
COPY GeoLite2-ASN.mmdb GeoLite2-City.mmdb /
