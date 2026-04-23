# Phase C: Application Architecture (AA)

## 1. Component Overview

### Backend (Spring Boot 3.x)

- **Root Package:** `com.bosch.shopfloor.supply`
- **Architecture Pattern:** Controller-Service-Repository

**Naming Conventions:**
- Controllers: `DeliveryOrderController.java`
- Services: `DeliveryOrderService.java`, `AuditLogService.java`
- Repositories: `DeliveryOrderRepository.java`, `AuditLogRepository.java`

**Rules:**
- All business logic, including state transition validation, **MUST** reside exclusively in the Service layer.
- Role-based access control (RBAC) **MUST** be enforced at the method level within the Service layer before any business logic is executed.
- The Controller layer is responsible only for HTTP request/response handling and DTO (Data Transfer Object) conversion.

---

### Frontend Standards (Angular 17+)

- **App Prefix:** `sfs` (Shopfloor Supply)
- **Styling:** Tailwind CSS for a utility-first approach.

**Naming Conventions:**
- Components: `order-dashboard.component.ts`, `create-order-form.component.ts`
- Services: `order-api.service.ts`

**Rules:**
- The UI **MUST** conditionally render actions (e.g., "Pick Up", "Receive" buttons) based on the user's role and the order's current state.
- The frontend **MUST NOT** contain any business logic for state transitions or permissions; it is purely a consumer of the backend API. All security and business rules are enforced by the backend.

---

## 2. Integration Contract

- **API Style:** RESTful
- **Base Path:** `/api/v1`
- **Format:** JSON (`application/json`) over HTTPS.
- **Authentication:** Bearer Token (JWT) must be passed in the `Authorization` header for all secured endpoints.
- **Status Codes:**
  - `200 OK`: Successful `GET` or `PATCH` request.
  - `201 Created`: Successful `POST` request.
  - `204 No Content`: Successful `DELETE` request.
  - `400 Bad Request`: The request is malformed or violates a business rule (e.g., invalid state transition). The response body **MUST** contain an error code and message.
  - `403 Forbidden`: The user's role does not permit this action.
  - `404 Not Found`: The requested resource (e.g., an order) does not exist.

---

## 3. Endpoints (Derived from Business Architecture)

This API contract is the single source of truth for client-server communication.

| Method | Path | Required Role | Description | Resulting State |
|---|---|---|---|---|
| `POST` | `/api/v1/orders` | Production Line User | Creates a new Delivery Order. | `NEW` |
| `GET` | `/api/v1/orders` | Any | Lists orders. Filters apply based on role. PL Users see their own; WH Users see `NEW` orders; Admins see all. | - |
| `GET` | `/api/v1/orders/{id}` | Any (owner or Admin) | Retrieves a single Delivery Order by its ID. | - |
| `PATCH` | `/api/v1/orders/{id}/status` | Varies | Updates the status of an order. The request body specifies the new status. | `IN_PREPARATION`, `IN_TRANSIT`, `COMPLETED` |
| `DELETE` | `/api/v1/orders/{id}` | Admin | Deletes a Delivery Order. A reason is required. | `DELETED` |
| `PUT` | `/api/v1/orders/{id}` | Admin | Manually edits an order's details or status. A reason is required. | Varies |

### State Transition Logic for `PATCH /api/v1/orders/{id}/status`
The service layer for this endpoint **MUST** enforce the state machine defined in Phase B.
- Request Body: `{ "status": "IN_PREPARATION" }` -> Requires **Warehouse User** role, current state must be `NEW`.
- Request Body: `{ "status": "IN_TRANSIT" }` -> Requires **Warehouse User** role, current state must be `IN_PREPARATION`.
- Request Body: `{ "status": "COMPLETED" }` -> Requires **Production Line User** role, current state must be `IN_TRANSIT`.

---

## 4. Application Pattern

- **Decoupled Client-Server:** The Angular Single-Page Application (SPA) is fully decoupled from the Spring Boot backend API. They communicate only via the RESTful JSON API defined above.
- **State Management (Frontend):** A state management library (e.g., NgRx or Akita) should be used in the Angular app to manage the state of orders and user sessions.

---

## 5. Interface Contract (API)
The API contract is fully defined in Section 3.

---

## 6. Security Implementation
- Security is a backend-first responsibility. RBAC **MUST** be implemented in the Spring Boot Service layer, for instance, using Spring Security's method-level security annotations (`@PreAuthorize`).
- The frontend **MUST NOT** be trusted. While UI elements will be hidden based on role, the backend API must re-validate every single request to ensure the authenticated user has the correct permissions.
- Roles **MUST** exactly match the definitions from Phase B: `ROLE_PRODUCTION`, `ROLE_WAREHOUSE`, `ROLE_ADMIN`.
