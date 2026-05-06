## Low-Level Design — Complete

**Requirement:** To create a web application for a shopfloor material supply system, allowing production line users to request materials from a warehouse. The system must track the order lifecycle from creation to delivery and provide administrators with override capabilities.

**Technology Stack:** Spring Boot (Java 17), PostgreSQL, Angular

### Generated Files

[x] Step 01: Data Model  → changes/01-data-model/proposal.md + tasks.md
[x] Step 02: Backend API → changes/02-backend-api/proposal.md + tasks.md
[x] Step 03: Frontend UI → changes/03-frontend-ui/proposal.md + tasks.md

### Layer Summaries

#### Data Model
The data model is designed with three core entities: `User`, `DeliveryOrder`, and `AuditLog`, all within a dedicated `shopfloor` schema. Key relationships are established between users and the orders they create or handle. The design includes soft-delete functionality for orders and a detailed audit trail for all administrative actions to ensure data integrity and traceability.

#### Backend API
The backend is built around a service-oriented architecture with three main services: `DeliveryOrderService`, `AuthService`, and `AuditLogService`. It exposes a RESTful API with six endpoints for managing the order lifecycle. Security is handled via JWT and method-level authorization with Spring Security's `@PreAuthorize` annotations, ensuring that business rules and role-based permissions are strictly enforced at the service layer.

#### Frontend UI
The frontend architecture is composed of five primary views and several reusable shared components. The design is role-centric, using Angular Guards to manage access to different routes and dynamically rendering UI elements based on the user's role and the state of the data. API integration is managed through dedicated services and a JWT interceptor, providing a clear and secure data flow from the UI to the backend.

### Cross-Cutting Concerns

- **Security:** Authentication is handled by JWTs. Authorization is enforced at both the route level (Angular Guards) and the service-method level (Spring Security), providing defense-in-depth.
- **Error Handling:** A global exception handler in the backend provides consistent JSON error responses. A frontend interceptor catches these errors and displays user-friendly toast notifications.
- **Audit Trail:** All administrative modifications are captured in an immutable `audit_logs` table, accessible only to administrators.
- **Data Integrity:** The system enforces the order lifecycle via a state machine in the service layer, preventing invalid status transitions. All data deletions are soft deletes.
