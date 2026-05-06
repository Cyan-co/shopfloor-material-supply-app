# Phase A: Architecture Vision

## 1. Vision and scope

- **Vision**: To create a seamless, real-time, and trackable digital workflow for material requests between the production floor and the warehouse, eliminating manual processes, reducing production delays, and providing a foundation for data-driven operational improvements.

- **Scope**: The initial scope (MVP) includes the core functionality for creating, accepting, preparing, and delivering material supply orders. It covers role-based access for Production Operators, Warehouse Staff, and Administrators. Key features include order creation, status tracking (Pending, Accepted, In-Transit, Delivered), and administrative capabilities for manual overrides and order management. Excluded from the initial scope are advanced features like proactive notifications, analytics dashboards, and direct inventory system integration.

- **Stakeholders**:
    - **Production Operators**: Users who create material supply requests from the shop floor.
    - **Warehouse Staff**: Users who fulfill the material requests.
    - **Administrators**: Users who manage the system, oversee the process, and handle exceptions.
    - **Production Managers**: Oversee production line efficiency and are key beneficiaries of reduced downtime.
    - **IT/Infrastructure Team**: Provide and maintain the necessary on-premises infrastructure.

---

## 2. Architecture principles (impact on implementation)

| # | Principle | Rationale | Implementation impact |
|---|---|---|---|
| 1 | **Stateless Services** | To ensure scalability and reliability, all backend services must be stateless. This allows for horizontal scaling and simplifies failover. | - No session data is to be stored in the memory of a service instance. <br>- All state must be externalized to a database or a dedicated cache. <br>- Every API request must be independent and contain all necessary information for processing. |
| 2 | **API-First Design** | The backend and frontend are decoupled. The API serves as the single source of truth and contract between them, enabling parallel development and future client expansion. | - The backend API must be designed and documented (using OpenAPI/Swagger) before frontend implementation begins. <br>- The frontend must only interact with the backend through the defined API endpoints. <br>- All business logic must reside in the backend, not in the frontend. |
| 3 | **Role-Based Access Control (RBAC)** | To enforce security and operational boundaries, user actions must be strictly controlled by their assigned roles (Operator, Warehouse, Admin). | - A centralized authentication and authorization service/module must be implemented. <br>- Every API endpoint must be protected and verify the user's role and permissions before processing the request. <br>- Frontend UI components must be conditionally rendered based on the user's role. |
| 4 | **Layered Architecture** | The application will be structured into distinct layers (Presentation, Business, Data Access) to ensure separation of concerns, making the system easier to maintain, test, and evolve. | - Code must be organized into corresponding modules/packages for each layer. (e.g., `controller`, `service`, `repository`). <br>- Dependencies must flow in one direction: Presentation -> Business -> Data Access. The Business layer must not know about the Presentation layer. |
| 5 | **Audit Logging for All State Changes** | To ensure traceability and accountability, every change to an order's state must be recorded in an immutable audit log. | - A dedicated `order_audit_log` table must be created in the database. <br>- Any service method that modifies an order's status (`create`, `accept`, `transit`, `deliver`, `edit`) must also write an entry to this log within the same transaction. <br>- The log entry must include the order ID, previous status, new status, timestamp, and the user who performed the action. |

---

## 3. How to use this document

- **When to read this document**: This document is the foundational guide for all subsequent design and development work. It should be read by all architects, developers, and QA engineers before starting any implementation task.
- **How to apply principles**: The principles listed above are not suggestions; they are mandatory constraints. All code, design decisions, and pull requests will be reviewed against these principles. For example, a pull request introducing a stateful service will be rejected.
- **How to resolve conflicts**: If a requirement seems to conflict with a principle, the conflict must be raised with the Solution Architect immediately. Principles can only be overridden with an explicit, documented exception in a formal Architecture Decision Record (ADR).

---

## 4. References

- Phase B – Business Architecture
- Phase C – Application & Data Architecture
- Phase D – Technology Architecture
- Phase F – Migration Plan
- Phase G – Governance
