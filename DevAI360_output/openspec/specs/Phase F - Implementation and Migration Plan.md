# Phase F: Implementation and Migration Plan

## 1. Implementation Overview

This plan outlines a 5-sprint (10-week) timeline to deliver the initial production-ready version of the Shopfloor Material Supply application.

### Development Methodology
- **Agile/Scrum:** The project will be executed in a series of 2-week sprints.
- **Release Cadence:** A release to STAGING will occur at the end of each sprint. The production release is planned at the end of Sprint 5.

### Project Timeline

| Phase | Sprints | Focus |
|---|---|---|
| **Phase 1: Foundation** | Sprint 1 | Infrastructure, CI/CD, Database Schema, App Scaffolding. |
| **Phase 2: Core Features** | Sprints 2-3 | Backend API and Frontend UI for the "golden path" workflow. |
| **Phase 3: Admin & Hardening**| Sprint 4 | Admin features, security hardening, performance testing. |
| **Phase 4: Deployment** | Sprint 5 | UAT, final testing, and production go-live. |

---

## 2. Sprint-Level Breakdown

### Sprint 1: Foundation
**Goal:** Establish the complete technical foundation. By the end of this sprint, a "hello world" application should be automatically deployable to the STAGING environment.
**Work Packages:** WP-01
**Deliverables:**
- Git repository with protected `main` branch.
- CI/CD pipeline in GitHub Actions that builds, tests, and deploys.
- Kubernetes manifests for STAGING.
- Initial PostgreSQL schema created via a Flyway script.
- Scaffolding for both Spring Boot and Angular applications.

### Sprints 2-3: Core Feature Development
**Goal:** Implement the primary workflow for Production Line and Warehouse users.
**Work Packages:** WP-02, WP-03, WP-04
**Deliverables:**
- **Backend:** All API endpoints for creating orders and transitioning them through `NEW`, `IN_PREPARATION`, `IN_TRANSIT`, and `COMPLETED` states are fully implemented and tested.
- **Frontend:** UI components are built for Production Line and Warehouse users to perform all actions in the "golden path."
- **Integration:** The frontend is fully integrated with the backend API for these core features.

### Sprint 4: Admin Features & System Hardening
**Goal:** Complete admin functionality and prepare the application for production-level quality.
**Work Packages:** WP-05
**Deliverables:**
- **Admin UI:** A dashboard for Admins to view all orders and perform manual edit/delete actions.
- **Observability:** Prometheus metrics and Grafana dashboards are configured.
- **Security:** Static code analysis (SAST) and dependency scanning are integrated into the pipeline. All critical vulnerabilities are addressed.
- **Testing:** End-to-end test suite is complete and passing.

### Sprint 5: UAT & Production Deployment
**Goal:** Validate the system with end-users and deploy to production.
**Deliverables:**
- Successful User Acceptance Testing (UAT) in the STAGING environment with sign-off from stakeholders.
- All production configurations (secrets, network policies) are in place.
- The application is deployed to the PROD Kubernetes environment.
- Post-deployment smoke tests are successfully executed.

---

## 3. Migration Strategy

### Migration Type
- **[x] Greenfield (no migration needed)**

This is a new application replacing a manual process, so there is no existing technical system or data to migrate.

---

## 4. Rollout Strategy

### Environment Progression
Deployment will be automated and will progress through environments based on Git flow:
`Feature Branch -> PR -> main (Deploy to STAGING) -> Git Tag (Deploy to PROD)`

### Rollback Procedures

| Scenario | Procedure | RTO (Recovery Time Objective) |
|---|---|---|
| Failed Production Deployment | Use Kubernetes native `kubectl rollout undo` command to revert to the previous stable deployment version. | < 5 minutes |
| Critical Bug Post-Launch | Rollback the deployment using the same procedure as a failed deployment. | < 5 minutes |
| Data Corruption | Restore the database from the last known good Point-in-Time Recovery (PITR) backup. | < 1 hour |

---

## 5. Success Metrics

### Technical Metrics

| Metric | Target | Measurement Tool |
|---|---|---|
| Availability | > 99.9% | Prometheus / Uptime Monitoring |
| API Response Time (p95) | < 500ms | Prometheus / Grafana |
| Code Coverage | > 80% | SonarQube |

### Business Metrics (from PRD)

| Metric | Target (within 6 months) | Measurement Method |
|---|---|---|
| Order Fulfillment Time | 30% reduction | Analytics on `delivery_orders` timestamps. |
| Incorrect Deliveries | 90% reduction | Tracking manual admin corrections in the `audit_logs`. |
| System Adoption | 100% of requests tracked | Application analytics. |

---

## 6. Milestones

| Milestone | Target | Criteria |
|---|---|---|
| M1: Foundation Complete | End of Sprint 1 | CI/CD pipeline is green; schema is deployed. |
| M2: Core Features Implemented | End of Sprint 3 | The primary user workflow is testable end-to-end in STAGING. |
| M3: Feature Complete | End of Sprint 4 | All features, including Admin, are complete. |
| M4: Production Go-Live | End of Sprint 5 | Application is live and available to all users in the PROD environment. |
