# Phase D: Technology Architecture (TA)

## 1. Infrastructure Overview

### Container Platform

- **Runtime:** Docker. All application components (backend, frontend) **MUST** be packaged as lightweight Docker images.
- **Orchestration:** Docker Compose will be used for local development environments. Kubernetes (K8s) is the target for STAGING and PROD environments to ensure scalability and resilience.
- **Registry:** All Docker images will be stored in a private container registry (e.g., GitHub Container Registry, Docker Hub Private, or a cloud provider's registry).

### Deployment Environments

| Environment | Purpose | Orchestration | Scaling |
|---|---|---|---|
| **DEV (Local)** | Individual developer environments | Docker Compose | 1 replica per service |
| **TEST** | Automated integration & E2E testing | Kubernetes | 1 replica per service |
| **STAGING** | Pre-production user acceptance testing (UAT) | Kubernetes | 2 replicas, production-like configuration |
| **PROD** | Live production environment | Kubernetes | 3+ replicas with Horizontal Pod Autoscaling (HPA) |

---

## 2. Network Architecture

### Ingress Configuration
- **Load Balancer:** A standard cloud load balancer will route traffic to the Kubernetes cluster.
- **Ingress Controller:** An NGINX Ingress Controller will be used within the K8s cluster to manage routing to the application services.
- **TLS Termination:** TLS **MUST** be terminated at the Ingress Controller. Certificates will be managed automatically using `cert-manager` with Let's Encrypt.
- **DNS:** A domain `sfs.yourcompany.com` will be pointed to the load balancer, with API traffic routed to `/api/*`.

### Service Communication
- **Internal:** All service-to-service communication within the Kubernetes cluster will use K8s DNS (e.g., `delivery-order-service.default.svc.cluster.local`).
- **Network Policies:** Kubernetes NetworkPolicies **MUST** be in place to restrict traffic. The frontend service should only be allowed to call the backend service, and the backend service should only be allowed to call the database.

---

## 3. Database Infrastructure

### PostgreSQL Configuration
- **Version:** 15+
- **Deployment:** A managed PostgreSQL service (e.g., AWS RDS, Azure Database for PostgreSQL) is highly recommended for production to handle backups, replication, and scaling.
- **Connection Pooling:** PgBouncer will be used as a sidecar container to manage database connection pools efficiently.
- **Backup Strategy:** Daily automated snapshots **MUST** be enabled, with Point-in-Time Recovery (PITR) configured for the last 7 days.
- **Replication:** A primary-replica setup **MUST** be used in the PROD environment for high availability.

---

## 4. Security Infrastructure

### Authentication & Authorization
- **Protocol:** OAuth 2.0 with OpenID Connect (OIDC) will be used.
- **Identity Provider (IdP):** An external IdP like Keycloak or a managed service (e.g., Auth0, Okta) will handle user authentication and token issuance. The application itself will not manage passwords.
- **Token Format:** JSON Web Tokens (JWT) will be used, containing user ID and roles.
- **Token Handling:** The backend will validate JWTs on every request. The frontend will store tokens securely (e.g., in memory or a secure storage mechanism, not `localStorage`).

### Secrets Management
- **Storage:** All secrets (database passwords, API keys, JWT signing keys) **MUST** be stored in Kubernetes Secrets.
- **Access:** Secrets will be mounted into pods as environment variables or files with strict, role-based access control (RBAC) policies.

---

## 5. Monitoring & Observability

### Metrics
- **Tool:** Prometheus will be used to scrape metrics from the backend service and database.
- **Dashboards:** Grafana dashboards will be created to visualize key metrics.
- **Required Metrics:**
  - **Application:** Request latency (p99), error rate (per endpoint), JVM heap usage.
  - **Business:** Delivery Orders created per hour, status transition times.

### Logging
- **Tool:** A Loki-based logging stack is the standard.
- **Log Format:** All application logs **MUST** be structured (JSON format) and include a `traceId` for correlating requests across services.

### Alerting
- **Tool:** Alertmanager will be configured with the following critical alerts:
  - **High Error Rate:** API error rate > 5% over a 5-minute period.
  - **Service Unavailability:** Application pods are down or not responding.
  - **Database Connection Failure:** The backend cannot connect to the PostgreSQL database.

---

## 6. CI/CD Pipeline

### Tooling
- **CI/CD:** GitHub Actions
- **Code Quality:** SonarQube integrated as a required check.

### Pipeline Stages
1.  **On Commit to Feature Branch:** `Build` (compile, run unit tests) -> `Code Quality Scan`.
2.  **On PR to `main`:** All previous stages + `Integration Test` -> `Build & Push Docker Image`.
3.  **On Merge to `main`:** `Deploy to STAGING`.
4.  **On Git Tag:** `Manual Approval` -> `Deploy to PROD`.

A deployment to production **MUST** require a manual approval step in GitHub Actions.

---

## 7. Disaster Recovery

### Backup Strategy
| Component | Frequency | Retention | RPO/RTO |
|---|---|---|---|
| **Database** | Daily Full, Continuous PITR | 30 days | RPO: < 5 mins, RTO: < 1 hour |
| **K8s Configs** | Stored in Git (IaC) | Infinite | RPO: N/A, RTO: < 30 mins |
| **Container Images**| Immutable | Per registry policy | RPO: N/A, RTO: < 30 mins |

### Recovery Procedures
- A documented runbook **MUST** be maintained for recovering the entire application stack from backups in a different region or cluster. The target Recovery Time Objective (RTO) for a full disaster is under 4 hours.
