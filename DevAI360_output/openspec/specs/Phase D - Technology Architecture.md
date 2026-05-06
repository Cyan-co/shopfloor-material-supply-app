# Phase D: Technology Architecture (TA)

## 1. Infrastructure Overview

### Container Platform

- **Runtime**: Docker will be used to containerize the Spring Boot backend and the Angular frontend (served via Nginx).
- **Orchestration**: Docker Compose will be used for local development (`DEV`) and small-scale `TEST` environments. Kubernetes is the target for `STAGING` and `PROD` environments to ensure scalability and resilience.
- **Registry**: A private container registry (e.g., Docker Hub Private, Harbor, or cloud provider equivalent) will store the application images.

### Deployment Environments

| Environment | Purpose | Orchestration | Scaling | Resources |
| :--- | :--- | :--- | :--- | :--- |
| **DEV** | Local developer machines | Docker Compose | 1 replica | Minimal |
| **TEST** | Automated integration and E2E testing | Docker Compose | 1 replica | Minimal |
| **STAGING** | Pre-production validation, user acceptance | Kubernetes | 2 replicas | Production-like |
| **PROD** | Live production environment | Kubernetes | 3+ replicas (HPA) | Full |

---

## 2. Network Architecture

### Ingress Configuration (Kubernetes)

- **Load Balancer**: A cloud-provider LoadBalancer service will be used, routing traffic to an NGINX Ingress Controller.
- **TLS Termination**: TLS will be terminated at the Ingress Controller. Certificates will be managed automatically using `cert-manager` with Let's Encrypt.
- **DNS**: The application will be accessible via a dedicated subdomain, e.g., `shopfloor-supply.your-company.com`.

### Service Communication

- **Internal**: Within the Kubernetes cluster, services will communicate using standard Kubernetes DNS names (e.g., `backend-service.default.svc.cluster.local`).
- **External**: All external traffic to the backend must pass through the Ingress Controller. Direct exposure of the backend service is forbidden.

### Network Policies

- **Default Deny**: A default-deny policy will be in place.
- **Allowed Traffic**:
    - Ingress Controller -> Frontend Service (Port 80)
    - Ingress Controller -> Backend Service (Port 8080)
    - Backend Service -> Database Service (Port 5432)

---

## 3. Database Infrastructure

### PostgreSQL Configuration

- **Version**: 15+
- **Connection Pooling**: PgBouncer will be used in front of the PostgreSQL instance in `STAGING` and `PROD` to manage connections efficiently.
- **Backup Strategy**: Daily automated snapshots with Point-In-Time Recovery (PITR) enabled, retained for 30 days.
- **Replication**: For `PROD`, a primary-replica setup will be used to ensure high availability for read operations and provide a hot standby.

---

## 4. Caching Infrastructure (if applicable)

### Redis Configuration

- **Purpose**: Initially out of scope for the MVP. If performance bottlenecks are identified, Redis will be introduced for caching user sessions and query results.
- **Eviction Policy**: `allkeys-lru` (Least Recently Used).

---

## 5. Security Infrastructure

### Authentication & Authorization

- **Protocol**: OAuth 2.0 (Authorization Code Flow).
- **Identity Provider (IdP)**: A dedicated IdP like Keycloak will be used. It will be responsible for user authentication and issuing JWTs.
- **Token Format**: JWT containing user ID, username, and role.
- **Token Storage**: The frontend will store tokens securely (e.g., in memory with refresh token logic).

### Secrets Management

- **Storage**: Kubernetes Secrets will be used to store all sensitive information (database passwords, API keys, JWT signing keys).
- **Access**: Secrets will be mounted into pods as environment variables or files with strict, role-based access control (RBAC) policies.

---

## 6. Monitoring & Observability

### Metrics

- **Tool**: Prometheus (via Spring Boot Actuator for the backend).
- **Dashboards**: Grafana dashboards will be created to visualize key metrics.
- **Required Metrics**: JVM metrics (heap, threads), HTTP request latency (p99), error rates (4xx, 5xx), database connection pool usage, and business metrics (orders created per hour).

### Logging

- **Tool**: ELK Stack (Elasticsearch, Logstash, Kibana).
- **Log Format**: All services must log in a structured JSON format.
- **Required Fields**: `timestamp`, `level`, `service_name`, `traceId`, `message`.

---

## 7. CI/CD Pipeline

### Pipeline Stages (GitHub Actions)

`Build` -> `Test` -> `Security Scan` -> `Publish Image` -> `Deploy to Staging` -> `Manual Approval` -> `Deploy to Prod`

### Stage Requirements

| Stage | Actions | Gate |
| :--- | :--- | :--- |
| **Build** | `mvn clean install` for backend, `npm run build` for frontend. Dockerize applications. | All compilation and unit tests pass. |
| **Test** | Run integration tests against a test database. | Code coverage > 80%. |
| **Security** | Run SAST (e.g., SonarQube) and dependency vulnerability scans. | No critical or high vulnerabilities found. |
| **Publish** | Push Docker images to the private container registry. | - |
| **Deploy** | Apply Kubernetes manifests for the target environment. | Previous stage success. |
| **Approval** | Manual gate for production deployment. | Sign-off from Dev Lead / Product Owner. |

---

## 8. Disaster Recovery

### Backup Strategy

| Component | Frequency | Retention | RTO | RPO |
| :--- | :--- | :--- | :--- | :--- |
| Database | Daily Snapshots | 30 days | < 1 hour | < 24 hours |
| K8s Configs | On change (Git) | Forever | < 15 mins | N/A |

### Recovery Procedures

- **Application Failure**: Kubernetes will automatically restart failed pods. A `helm rollback` command can be used to revert to a previous stable deployment.
- **Database Failure**: The database will be restored from the latest successful snapshot.
- **Full Environment Disaster**: The entire environment will be recreated from the Infrastructure as Code (IaC) scripts and application manifests stored in Git.
