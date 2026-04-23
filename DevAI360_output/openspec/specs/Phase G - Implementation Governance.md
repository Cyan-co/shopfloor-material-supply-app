# Phase G: Implementation Governance

## 1. Governance Overview

### Governance Objectives
- **Ensure Compliance:** To guarantee that the implemented solution adheres strictly to the defined architecture documents (Phases A-F).
- **Prevent Architecture Drift:** To use automated checks and clear review processes to prevent gradual deviation from the architectural vision.
- **Enable Agility:** To create a governance framework that is lightweight and automated, enabling development teams to move quickly without compromising quality or standards.

---

## 2. Architecture Compliance Criteria

### 2.1 Code Architecture Compliance

| Criterion | Rule | Verification Method |
|---|---|---|
| **Layer Separation** | The Controller layer must not directly access the Repository layer. All business logic must flow through the Service layer. | **ArchUnit Test** in the CI pipeline. |
| **Package Structure** | All backend code must reside under the `com.bosch.shopfloor` package. | Static analysis (Linting/Checkstyle). |
| **Dependency Control** | No new third-party libraries may be added without approval. | Dependency scanning (e.g., Trivy) and pull request reviews. |

### 2.2 API Compliance

| Criterion | Rule | Verification Method |
|---|---|---|
| **API Contract** | The implemented API must match the OpenAPI specification derived from the Application Architecture. | **Contract Testing** in the CI pipeline. |
| **Security** | All API endpoints (except for a public health check) must enforce role-based access control. | Automated security scan (DAST) in the pipeline. |

### 2.3 Data Compliance

| Criterion | Rule | Verification Method |
|---|---|---|
| **Schema Changes** | All database schema changes **MUST** be performed via a new, versioned Flyway migration script. | CI pipeline fails if manual schema changes are detected. |
| **Index Mandate** | All foreign key columns **MUST** be indexed. | A script run during the CI pipeline will analyze the schema and fail the build if an un-indexed FK is found. |

---

## 3. Automated Enforcement

The CI/CD pipeline is the primary mechanism for enforcing architecture governance.

### 3.1 CI/CD Pipeline Gates

| Gate | Stage | Failure Action |
|---|---|---|
| **Unit & ArchUnit Tests** | Build | **Block Merge** |
| **SonarQube Quality Gate** | Build | **Block Merge** (if code coverage < 80% or new critical bugs) |
| **SAST Security Scan** | Build | **Block Merge** (if new critical vulnerabilities are found) |
| **Contract Tests** | Test (Post-deployment to TEST env) | **Block Deployment** to STAGING |
| **End-to-End Tests** | Test | **Block Deployment** to STAGING |

### 3.2 Quality Gates (SonarQube)

| Metric | Threshold |
|---|---|
| Code Coverage | >= 80% |
| Duplicated Lines | <= 3% |
| Security Vulnerabilities | 0 Critical or High |
| Maintainability Rating | 'A' |

---

## 4. Review Processes

### 4.1 Code Review Requirements
- **Standard Changes:** Require at least one peer developer approval.
- **Critical Changes:** (e.g., API contract, DB schema, security logic) Require approval from both a peer and the Tech Lead.

### 4.2 Architecture Decision Records (ADR)
An ADR is mandatory for any intentional deviation from the established architecture.
- **Process:** The developer proposing the change writes an ADR detailing the context, decision, and consequences. The ADR is then reviewed by the Tech Lead and Solution Architect.
- **Storage:** ADRs are stored as markdown files in the `/docs/adr` directory of the Git repository.

---

## 5. Exception Handling

Deviations from the architecture are strongly discouraged. When necessary, they must be handled through the formal ADR process. A decision to deviate must be justified by a significant, unforeseen technical or business constraint.

---

## 6. Roles and Responsibilities

| Role | Responsibilities in Governance |
|---|---|
| **Developer** | Adhere to the architecture, write compliant code, write ADRs for proposed deviations. |
| **Tech Lead** | Enforce standards during code review, approve critical changes, review ADRs. |
| **Solution Architect**| Define the initial architecture, review and approve ADRs, periodically review the project for architecture drift. |
| **DevOps Engineer** | Maintain and enforce the automated gates within the CI/CD pipeline. |
