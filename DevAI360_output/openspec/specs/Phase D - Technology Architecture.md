# Phase D: Technology Architecture (TA)

## 1. Infrastructure Overview

### Container Platform
- **Runtime**: Docker will be used to containerize the Spring Boot backend and the Angular frontend (served via Nginx).
- **Orchestration**:
    - **Local/Dev**: Docker Compose for simplicity and fast iteration.
    - **Staging/Prod**: Kubernetes for scalability, resilience, and automated management.
- **Registry**: A private container registry (e.g., GitHub Container Registry, Docker Hub Private, or a cloud provider's registry) will be used to store the Docker images.

### Deployment Environments
| Environment | Purpose | Orchestration | Scaling |
|---|---|---|---|
| **Local** | Individual developer machines | Docker Compose | 1 replica per service |
| **Dev** | Shared integration environment | Kubernetes | 1 replica per service |
| **Staging** | Pre-production validation, UAT | Kubernetes | 2 replicas, production-like configuration |
| **Prod** | Live production environment | Kubernetes | 3+ replicas with Horizontal Pod Autoscaler (HPA) |

---

## 2. Network Architecture

### Ingress Configuration
- **Load Balancer**: A cloud-native load balancer (e.g., AWS ALB, Google Cloud Load Balancer) will be managed by a Kubernetes Ingress Controller (e.g., NGINX Ingress).
- **TLS Termination**: TLS will be terminated at the Ingress Controller. Certificates will be managed automatically using `cert-manager` with Let's Encrypt.
- **DNS**: The application will be accessible via a standard DNS structure (e.g., `shopfloor-app.example.com`).

### Service Communication
- **Internal**: Services within the Kubernetes cluster will communicate using standard Kubernetes DNS service names (e.g., `backend-service.default.svc.cluster.local`).
- **External**: All external traffic to the backend API will be routed through the Ingress Controller, which acts as the single API Gateway.

### Network Policies
Kubernetes NetworkPolicies will be used to enforce a "zero-trust" network.
- By default, all pod-to-pod communication is denied.
- Explicit policies will be created to allow traffic:
    - From the Ingress Controller to the backend service on its designated port.
    - From the backend service to the PostgreSQL database on port 5432.

---

## 3. Database Infrastructure

### PostgreSQL Configuration
- **Version**: 15+
- **Deployment**: A managed PostgreSQL service from a cloud provider (e.g., AWS RDS, Google Cloud SQL) is strongly recommended for production to handle backups, replication, and failover automatically.
- **Connection Pooling**: `PgBouncer` will be used as a sidecar container to the backend application pods to manage an efficient pool of connections to the database.
- **Backup Strategy**: Daily automated snapshots with Point-in-Time Recovery (PITR) enabled, retained for 30 days.
- **Replication**: For the PROD environment, a primary-replica setup will be used to ensure high availability.

---

## 4. Caching Infrastructure
- **Redis**: A managed Redis instance will be used for caching non-critical, frequently accessed data to reduce database load.
- **Use Cases**:
    - Caching user sessions or JWT metadata.
    - Caching results of read-heavy, non-volatile API endpoints.
- **Eviction Policy**: `allkeys-lru` (Least Recently Used).

---

## 5. Security Infrastructure

### Authentication & Authorization
- **Protocol**: OAuth 2.0 (Authorization Code Flow for the frontend).
- **Identity Provider (IdP)**: A dedicated IdP like Keycloak or a cloud-native service (e.g., AWS Cognito) will manage user identities and issue JWTs. This externalizes user management from the application code.
- **Token Format**: JWT containing standard claims (`sub`, `exp`, `iat`) and custom claims for `user_id` and `role`.

### Secrets Management
- **Storage**: Kubernetes Secrets will be used to store all sensitive information (database passwords, API keys, JWT signing keys).
- **Access**: Secrets will be mounted into pods as environment variables or files at runtime. The principle of least privilege will be applied via Kubernetes RBAC.

### TLS Configuration
- **Minimum Version**: TLS 1.2 is required for all external and internal communication.
- **Certificates**: Managed by `cert-manager` for public-facing endpoints.

---

## 6. Monitoring & Observability

### Metrics
- **Tool**: Prometheus will be deployed to the Kubernetes cluster to scrape metrics.
- **Dashboards**: Grafana will be used to visualize key metrics.
- **Required Metrics**:
    - **Backend**: JVM metrics (heap, threads), HTTP request latency (p95, p99), error rates per endpoint, database connection pool usage.
    - **Database**: CPU utilization, active connections, query throughput.
    - **Business**: Number of orders created per minute, status transition counts.

### Logging
- **Tool**: The ELK Stack (Elasticsearch, Logstash, Kibana) or a similar solution like Loki/Grafana will be used for centralized log aggregation.
- **Log Format**: All application logs will be structured (JSON format) and written to `stdout`.
- **Required Fields**: `timestamp`, `level`, `service_name`, `trace_id`, `message`.

### Alerting
- **Tool**: Alertmanager (part of the Prometheus ecosystem) will be configured to send alerts to a designated channel (e.g., Slack, PagerDuty).
- **Key Alerts**:
    - High API Error Rate (>5% in 5 minutes).
    - High API Latency (p99 > 1.5s).
    - Kubernetes Pod Crash Looping.
    - Database CPU Utilization > 80%.

---

## 7. CI/CD Pipeline

### Pipeline Stages (GitHub Actions)
`Push to Feature Branch` -> `Build & Test` -> `Push to Dev` -> `Merge to Main` -> `Deploy to Staging` -> `Manual Approval` -> `Deploy to Prod`

| Stage | Actions | Gate |
|---|---|---|
| **Build & Test** | - Run `mvn clean install` <br> - Execute unit & integration tests <br> - Build Docker image | All tests must pass with >80% code coverage. |
| **Security Scan** | - SonarQube static analysis (SAST) <br> - Dependency vulnerability scan (e.g., Snyk) | No critical or high-severity vulnerabilities found. |
| **Deploy to Staging** | Automatically deploy the new image to the Staging Kubernetes environment. | Successful merge to the `main` branch. |
| **Deploy to Prod** | A manual approval step in GitHub Actions is required. | Successful automated tests against Staging + manual sign-off. |

---

## 8. Disaster Recovery

### Backup Strategy
| Component | Frequency | Retention | RTO (Recovery Time Objective) |
|---|---|---|---|
| **Database (PostgreSQL)** | Daily full snapshot, continuous PITR | 30 days | < 1 hour |
| **Container Images** | Immutable, stored in registry | Indefinitely | N/A (re-deployable) |
| **Kubernetes Config (IaC)**| Stored in Git | Indefinitely | < 30 minutes |

### Recovery Procedures
- **Application Failure**: Kubernetes will automatically restart failed pods. A failed deployment can be instantly rolled back using `kubectl rollout undo`.
- **Database Failure**: The managed database service will automatically failover to the read replica. For data corruption, a point-in-time recovery from the backups will be initiated.
- **Full Regional Disaster**: A documented runbook will detail the process to recreate the entire infrastructure in a different region from the Infrastructure-as-Code (IaC) templates and database backups.

---
