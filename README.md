# Consiva.ai — Kubernetes deployment configuration

Deployment configuration for running **Consiva.ai**, a DPDP-compliant cookie and consent
management platform, inside your own Kubernetes cluster. This is the configuration published
alongside the Consiva.ai listing on Google Cloud Marketplace.

Deployed this way, the application runs entirely in your infrastructure: consent records and
data-principal requests never leave your cluster, and the application makes no outbound call to
Consiva at runtime.

**Deploying? Start with [`USER_GUIDE.md`](USER_GUIDE.md).** It is the complete installation guide.

---

## Layout

| Path | Purpose |
|---|---|
| `chart/` | Helm chart — Deployment (backend + frontend), Services, Ingress, GKE `BackendConfig`, ConfigMap, Secret, Application resource |
| `deployer/Dockerfile` | Marketplace deployer image, built `FROM gcr.io/cloud-marketplace-tools/k8s/deployer_helm` |
| `schema.yaml` | Deployer input schema shown in the Marketplace deployment form |
| `apptest/chart/templates/tests.yaml` | Verification Pod run after install |
| `build-images.sh` | Builds, tags and publishes the images |
| `USER_GUIDE.md` | Installation and operations guide |
| `LICENSE` | Apache-2.0 |

Current release: **1.0.8**.

## What you deploy

One Pod with two containers — the API backend and the administrator dashboard — two Services,
an optional Ingress, a ConfigMap, a Secret and an Application resource. Full component table and
path-routing rules are in the user guide.

## Database

**This chart ships no database.** `db.connectionString` is a required input; point it at Cloud SQL
for SQL Server or at any SQL Server instance your cluster can reach. Consent records are your own
regulated data, so they stay on storage you control, under your own backup and retention policy.

The connection string must include `;IPAddressPreference=IPv4First` — it works around a
`Microsoft.Data.SqlClient` fault resolving IP addresses on Linux containers. Without it the
application cannot reach your database.

## Licensing and tiers

- **Blank licence key** → Free tier. Fully functional, no contract required.
- **Valid signed key** → Enterprise until the key's expiry date.

The signature is verified once at startup and the expiry re-checked on every read, so a
long-running container moves back to the Free tier the moment the term ends, with no restart and
no interruption. Verification is entirely offline — nothing is sent to Consiva or to Google.

Enterprise keys are issued by Consiva directly. Use **Contact Sales** on the Marketplace listing.

## Ingress

The Ingress is **opt-in** (`ingress.enabled: false` by default), because on GKE it provisions an
external Application Load Balancer that can take several minutes to become reachable. The
Services are usable over in-cluster DNS immediately.

`BackendConfig` is a GKE-only resource and is gated behind `ingress.backendConfig` (default
`true`, read only when `ingress.enabled` is true). **On a non-GKE cluster set it to `false`**, or
the install fails with `no matches for kind "BackendConfig"`. See the user guide.

## Health endpoints

| Endpoint | Meaning |
|---|---|
| `/health` | Liveness. Returns 200 whenever the process is running, including while the application is still converging on a database that is not yet reachable. |
| `/health/ready` | Readiness. Returns 503 until the schema and seed data are in place. |

**Point your monitoring at `/health/ready`.**

## Support

- Issues with this configuration: open an issue on this repository.
- Product, licensing and Enterprise enquiries: **Contact Sales** on the Google Cloud Marketplace
  listing, or <support@consiva.ai>.

## Licence

Apache-2.0 — see [`LICENSE`](LICENSE).
