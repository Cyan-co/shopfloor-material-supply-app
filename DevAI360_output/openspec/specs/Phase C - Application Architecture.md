# Phase C: Application Architecture (AA)

## 1. Component Overview

This architecture mandates a decoupled Single-Page Application (SPA) model. The frontend and backend are separate applications that communicate via a RESTful API.

### Backend (Spring Boot 3.x)

- **Language:** Java 21
- **Framework:** Spring Boot 3.x
- **Root Package:** `com.bosch.shopfloor`
- **Architecture Pattern:** A strict three-tier Controller-Service-Repository pattern is mandatory.
  - **Controllers:** Handle HTTP requests, deserialize inputs, and serialize outputs. No business logic is permitted here.
  - **Services:** Contain all business logic, transaction management, and security enforcement (RBAC). This is the core application layer.
  - **Repositories:** Responsible for data access and persistence.

#### Naming Conventions:
- Controllers: `DeliveryOrderController.java`, `AdminController.java`
- Services: `DeliveryOrderService.java`, `AuditLogService.java`
- Repositories: `DeliveryOrderRepository.java`, `AuditLogRepository.java`

#### Rules:
- All business rules and state transition logic defined in Phase B **MUST** be implemented exclusively in the Service layer.
- Role-based access control (RBAC) checks **MUST** be performed within the Service layer methods before executing any business logic.

---

### Frontend (Angular 17+)

- **Framework:** Angular 17+
- **Styling:** Tailwind CSS for a utility-first styling approach.
- **App Prefix:** `app-shopfloor`
- **Architecture:** The application will be structured into modules corresponding to user roles (e.g., `production-line`, `warehouse`, `admin`).

#### Naming Conventions:
- Components: `order-dashboard.component.ts`, `create-order-form.component.ts`
- Services: `order-api.service.ts` (for communicating with the backend API)

#### Rules:
- The UI **MUST** dynamically render components and actions (e.g., show/hide buttons) based on the authenticated user's role.
- The frontend **MUST NOT** contain any business rule logic. It is only responsible for presentation and user interaction. All business decisions are made by the backend.

---

## 2. Integration Contract

- **API Style:** RESTful
- **Base Path:** `/api`
- **Data Format:** JSON (`application/json`) will be used for all request and response bodies.
- **Authentication:** Stateless token-based authentication (e.g., JWT) will be used. The token will contain the user's ID and role.

---

## 3. Endpoints (Derived from Business Architecture)

The following endpoints are derived directly from the business process defined in Phase B.

| Method | Endpoint | Allowed Roles | Description |
|---|---|---|---|
| `POST` | `/api/orders` | `Production Line User` | Creates a new Delivery Order. The request body contains `materialName`, `quantity`, and `deliveryLocation`. |
| `GET` | `/api/orders` | All Roles | Lists orders. Results are filtered based on role: Production Line User sees their own; Warehouse User sees 'NEW'; Admin sees all. |
| `GET` | `/api/orders/{id}` | All Roles | Retrieves a single Delivery Order by its ID. |
| `PATCH` | `/api/orders/{id}/status` | `Warehouse User`, `Production Line User`, `Admin` | Updates the status of an order. The request body is `{"status": "NEW_STATUS"}`. The backend service will enforce which roles can transition to which states. |
| `DELETE` | `/api/orders/{id}` | `Admin` | Deletes a Delivery Order. A reason for deletion must be logged. |

---

## 4. Security Implementation

- **Enforcement Layer:** Security, especially Role-Based Access Control (RBAC), **MUST** be enforced in the **backend Service layer**.
- **Trust Model:** The frontend is considered an untrusted client. No security decisions should be made based on information coming from the frontend without backend verification.
- **Roles:** The roles used for security checks (`Production Line User`, `Warehouse User`, `Admin`) **MUST** directly correspond to the roles defined in the Phase B Business Architecture. These roles will be embedded in the user's authentication token.
