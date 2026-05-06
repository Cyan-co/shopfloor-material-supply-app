# Proposal: Shopfloor Material Supply System Frontend UI

**Ref:** Depends on `@openspec/changes/01-data-model`, `@openspec/changes/02-backend-api`

This document defines the complete frontend design: view architecture, component breakdown, user flows per role, role-based visibility rules, and the API integration map.

---

## 1. Application Routes

| Route | Component | Guard | Accessible To |
|---|---|---|---|
| `/login` | `LoginComponent` | - | Unauthenticated Users |
| `/dashboard`| `OrderDashboardComponent` | `AuthGuard` | All Authenticated |
| `/orders/new`| `CreateOrderFormComponent` | `AuthGuard`, `RoleGuard('PRODUCTION_OPERATOR')` | `PRODUCTION_OPERATOR` |
| `/admin` | `AdminDashboardComponent`| `AuthGuard`, `RoleGuard('ADMINISTRATOR')` | `ADMINISTRATOR` |
| `/admin/orders/:id/history` | `AuditLogComponent` | `AuthGuard`, `RoleGuard('ADMINISTRATOR')` | `ADMINISTRATOR` |
| `**` | `NotFoundComponent` | - | All Users |

---

## 2. View Architecture

### `LoginComponent`
- **Route:** `/login`
- **Purpose:** Authenticates users and stores the JWT.
- **Layout:** Centered form with username, password, and a login button. Displays error messages on failure.

### `OrderDashboardComponent`
- **Route:** `/dashboard`
- **Purpose:** The main view for all users, displaying a list of delivery orders filtered by their role.
- **Layout:**
  - Header with a "New Order" button (for Production Operators).
  - A data table of orders with columns for ID, material, quantity, status, and actions.
  - Status is displayed using a color-coded `StatusBadgeComponent`.
  - Action buttons are rendered conditionally based on the user's role and the order's status.
- **Data Source:** `GET /api/orders` (the backend handles role-based filtering).
- **Polling:** The order list will refresh automatically every 30 seconds.

### `CreateOrderFormComponent`
- **Route:** `/orders/new`
- **Purpose:** A form for Production Operators to create new material requests.
- **Layout:** A reactive form with fields for all required order details. Includes validation and error messages.

### `AdminDashboardComponent`
- **Route:** `/admin`
- **Purpose:** Provides Administrators with a complete overview and control over all orders.
- **Layout:**
    - A comprehensive data table showing all orders.
    - Controls to manually edit an order's status or delete an order.
    - A link to view the audit history for each order.

### `AuditLogComponent`
- **Route:** `/admin/orders/:id/history`
- **Purpose:** Displays the audit trail for a specific order.
- **Layout:** A read-only table listing all administrative actions for the order.

---

## 3. User Flows

### Flow 1: Production Operator
1.  Logs in and is directed to `/dashboard`.
2.  Views their active orders.
3.  Clicks "New Order" to navigate to `/orders/new`.
4.  Fills out and submits the form.
5.  Is redirected to the dashboard and sees the new order with `PENDING` status.
6.  When an order's status becomes `IN_TRANSIT`, a "Receive" button appears.
7.  Clicks "Receive" to finalize the order, changing its status to `DELIVERED`.

### Flow 2: Warehouse Staff
1.  Logs in and is directed to `/dashboard`.
2.  Views a queue of `PENDING` orders.
3.  Clicks "Accept" on an order to change its status to `ACCEPTED`. The "Accept" button is replaced with an "In Transit" button.
4.  Clicks "In Transit" to change the status to `IN_TRANSIT`.

### Flow 3: Administrator
1.  Logs in and is directed to `/admin` or a unified dashboard with admin controls.
2.  Views all orders.
3.  Can change any order's status via a dropdown.
4.  Can delete any order, which triggers a confirmation dialog.
5.  Can navigate to the audit history for any order.

---

## 4. Role-Based Visibility Matrix

| UI Element | `PRODUCTION_OPERATOR` | `WAREHOUSE_STAFF` | `ADMINISTRATOR` |
|---|---|---|---|
| "New Order" Button | ✓ | ✗ | ✓ |
| "Accept" Button | ✗ | ✓ (on `PENDING`) | ✓ |
| "In Transit" Button | ✗ | ✓ (on `ACCEPTED`)| ✓ |
| "Receive" Button | ✓ (on `IN_TRANSIT`)| ✗ | ✓ |
| "Delete" Button | ✗ | ✗ | ✓ |
| Status Override | ✗ | ✗ | ✓ |
| View Audit History | ✗ | ✗ | ✓ |

---

## 5. Component Breakdown

| Component | File Location | Purpose |
|---|---|---|
| `LoginComponent` | `components/login/` | Handles authentication. |
| `OrderDashboardComponent` | `components/order-dashboard/`| Main view for orders. |
| `CreateOrderFormComponent`| `components/create-order-form/`| New order submission. |
| `AdminDashboardComponent` | `components/admin-dashboard/` | Admin overview and controls. |
| `AuditLogComponent` | `components/audit-log/` | Displays audit history. |
| `StatusBadgeComponent` | `shared/components/status-badge/` | Reusable colored status chip. |
| `ConfirmDialogComponent` | `shared/components/confirm-dialog/`| Modal confirmation prompt. |
| `NavbarComponent` | `layout/navbar/` | Top navigation with role-based links. |

---

## 6. API Integration Map

| Component | Service Method | HTTP | Endpoint |
|---|---|---|---|
| `LoginComponent` | `AuthService.login()` | POST | `/api/auth/login` |
| `OrderDashboardComponent`| `OrderApiService.getOrders()` | GET | `/api/orders` |
| `OrderDashboardComponent`| `OrderApiService.updateOrderStatus()` | PATCH| `/api/orders/{id}/status` |
| `CreateOrderFormComponent`| `OrderApiService.createOrder()` | POST | `/api/orders` |
| `AdminDashboardComponent`| `OrderApiService.deleteOrder()` | DELETE| `/api/orders/{id}` |
| `AuditLogComponent` | `AdminApiService.getAuditLogs()` | GET | `/api/orders/{id}/history` |

---

## 7. UX Principles

- **Clarity:** The status of every order will be immediately clear through color-coded badges.
- **Role-based UI:** Users will only see the controls and data relevant to their role.
- **Feedback:** All actions will provide immediate feedback (e.g., loading spinners, success/error toasts).
