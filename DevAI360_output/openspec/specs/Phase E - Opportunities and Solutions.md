# Phase E: Opportunities and Solutions

## 1. Gap Analysis Summary

### Current State
**Greenfield.** There is no existing automated system. The current process is manual, relying on verbal communication or paper-based requests, which is inefficient and error-prone.

### Target State
The target state is the fully implemented system as defined in the **Phase B (Business)**, **Phase C (Application & Data)**, and **Phase D (Technology)** architecture documents. This includes a containerized, cloud-native web application with a clear business process, a robust data model, and a secure, observable infrastructure.

### Gap Summary Table
| # | Gap Area | Current State | Target State | Priority |
|---|---|---|---|---|
| 1 | Core Application | No System | A fully functional web application (frontend and backend) that implements the defined business logic and API. | High |
| 2 | Data Persistence | No Database | A structured PostgreSQL database with the specified schema and data integrity rules. | High |
| 3 | Infrastructure | No Infrastructure | A containerized deployment on Kubernetes across DEV, STAGING, and PROD environments. | High |
| 4 | Automation | Manual Process | A full CI/CD pipeline for automated builds, testing, and deployments. | High |
| 5 | Observability | No Visibility | A comprehensive monitoring and logging solution (Prometheus, Grafana, Loki). | Medium |

---

## 2. Solution Building Blocks

### 2.1 Application Building Blocks
| ID | Building Block | Description | Gap Addressed | Build/Buy |
|---|---|---|---|---|
| **ABB-01** | Backend API (Spring Boot) | The core RESTful API service that enforces all business logic, state transitions, and security. | Gap #1 | Build |
| **ABB-02** | Frontend Web App (Angular) | The single-page application (SPA) that provides the user interface for all roles. | Gap #1 | Build |
| **ABB-03** | Authentication Service Integration | Client-side and server-side logic to integrate with the chosen OAuth 2.0/OIDC Identity Provider. | Gap #1 | Buy/Reuse |

### 2.2 Data Building Blocks
| ID | Building Block | Description | Gap Addressed | Build/Buy |
|---|---|---|---|---|
| **DBB-01** | Database Schema (PostgreSQL) | The set of tables, columns, indexes, and constraints defined in the Data Architecture. Implemented via Flyway migrations. | Gap #2 | Build |

### 2.3 Technology Building Blocks
| ID | Building Block | Description | Gap Addressed | Build/Buy |
|---|---|---|---|---|
| **TBB-01** | Containerization | Dockerfiles for the backend and frontend applications. | Gap #3 | Build |
| **TBB-02** | Infrastructure as Code (IaC) | Kubernetes manifests (or Helm charts) to define all environments (DEV, STAGING, PROD). | Gap #3, #4 | Build |
| **TBB-03** | CI/CD Pipeline | The GitHub Actions workflow definitions for the entire build, test, and deploy process. | Gap #4 | Build |
| **TBB-04** | Monitoring & Logging Stack | Configuration for Prometheus, Grafana, and Loki to collect and display application metrics and logs. | Gap #5 | Buy/Reuse |

---

## 3. Build vs. Buy Analysis

### Build Decisions
| Component | Rationale |
|---|---|
| **Backend & Frontend Apps** | The core business logic is unique to the shopfloor process. A custom build provides the exact required functionality and flexibility for future expansion without vendor lock-in. |
| **Database Schema** | The data model is specific to this application's needs. |
| **IaC & CI/CD Pipeline** | These are custom configurations that define our specific deployment and testing strategy. |

### Buy/Reuse Decisions
| Component | Product/Library | Rationale |
|---|---|---|
| **Database** | Managed PostgreSQL Service | A managed service offloads the operational burden of backups, patching, and high availability, which is more cost-effective than self-hosting. |
| **Identity Provider** | Keycloak / Auth0 / Okta | These are mature, secure, and feature-rich identity solutions. Building a custom IdP is complex and unnecessary. |
| **Monitoring Stack** | Prometheus, Grafana, Loki | These are industry-standard, open-source tools that are powerful and well-supported. |
| **Web & API Frameworks**| Angular, Spring Boot | These are powerful open-source frameworks that accelerate development significantly. |

---

## 4. Work Packages

### Work Package Definition
| WP ID | Name | Building Blocks | Dependencies | Estimated Effort |
|---|---|---|---|---|
| **WP-01** | Foundational Setup | DBB-01, TBB-01, TBB-02, TBB-03 | - | 1 Sprint |
| **WP-02** | Core API Implementation | ABB-01 (partial), ABB-03 | WP-01 | 2 Sprints |
| **WP-03** | Core Frontend Implementation | ABB-02 (partial) | WP-02 | 2 Sprints |
| **WP-04** | Admin & Monitoring | ABB-01/02 (Admin features), TBB-04 | WP-03 | 1 Sprint |

### Work Package Sequencing
A phased approach is recommended to deliver value incrementally.

`WP-01 (Foundation: CI/CD, DB Schema, K8s Config)`
    `↓`
`WP-02 (Backend: Implement core API endpoints and security)`
    `↓`
`WP-03 (Frontend: Build UI for Production & Warehouse users)`
    `↓`
`WP-04 (Admin & Ops: Build Admin UI and configure monitoring)`

---

## 5. Risk Assessment
| Risk | Impact | Probability | Mitigation |
|---|---|---|---|
| User Adoption | Medium | Medium | Involve key users from each role during the development of the UI/UX in WP-03 and WP-04. Provide training before go-live. |
| Network Reliability | High | Low | The application is designed for a reliable network. If shopfloor connectivity is poor, this could impact usability. A network assessment should be conducted. |
| Scope Creep | Medium | High | Adhere strictly to the requirements defined in the PRD and architecture documents. All changes must go through a formal change request process. |
