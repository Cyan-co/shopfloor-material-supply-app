# Proposal: Shopfloor Material Supply App Backend API

**Ref:** Depends on `@openspec/changes/01-data-model`

## 1. Service Layer

### AuthService
- **Description**: Handles user authentication and session management.
- **Methods**:
    - `login(username, password)`: Authenticates a user and returns a session token (e.g., JWT).
    - `logout(token)`: Invalidates a user's session.

### OrderService
- **Description**: Manages the core business logic for delivery orders.
- **Methods**:
    - `createOrder(orderData, requestorUser)`: Creates a new `DeliveryOrder` and an initial `AuditLog` entry.
    - `getOrdersByStatus(status)`: Fetches orders filtered by a specific status.
    - `getAllOrders(filters)`: Fetches all orders, with optional filters for admins.
    - `pickupOrder(orderId, warehouseUser)`: Assigns a warehouse user and transitions status from `NEW` to `IN_PREPARATION`.
    - `markOrderInTransit(orderId, warehouseUser)`: Transitions status from `IN_PREPARATION` to `IN_TRANSIT`.
    - `receiveOrder(orderId, productionUser)`: Transitions status from `IN_TRANSIT` to `COMPLETED`.
    - `editOrderStatus(orderId, newStatus, reason, adminUser)`: Allows an admin to manually change an order's status.
    - `deleteOrder(orderId, reason, adminUser)`: Allows an admin to delete an order.

### AuditService
- **Description**: Handles the creation of audit log entries.
- **Methods**:
    - `logAction(orderId, action, details, user)`: Creates a new `AuditLog` entry. This will be called by the `OrderService` methods.

## 2. Business Rules

1. **Order Creation**:
   - **When**: A user with the `PRODUCTION_LINE` role submits a new order request.
   - **Then**: A `DeliveryOrder` is created with status `NEW`. An audit log is created.

2. **Order Pickup**:
   - **When**: A user with the `WAREHOUSE` role "picks up" an order with `NEW` status.
   - **Then**: The order status changes to `IN_PREPARATION` and the `warehouse_user_id` is set. An audit log is created.

3. **Order Transit**:
   - **When**: The assigned warehouse user marks an `IN_PREPARATION` order for transit.
   - **Then**: The order status changes to `IN_TRANSIT`. An audit log is created.

4. **Order Completion**:
   - **When**: The original `requestor_id` user "receives" an `IN_TRANSIT` order.
   - **Then**: The order status changes to `COMPLETED`. An audit log is created.

5. **Admin Override**:
   - **When**: A user with the `ADMIN` role edits or deletes an order.
   - **Then**: The action is performed, and an audit log is created with the provided reason.

## 3. API Endpoints

| Method | Path | Description | Auth Required | Role(s) |
|---|---|---|---|---|
| POST | /api/v1/auth/login | Authenticate and get a session token. | No | - |
| POST | /api/v1/auth/logout | Invalidate the current session token. | Yes | All |
| POST | /api/v1/orders | Create a new delivery order. | Yes | `PRODUCTION_LINE` |
| GET | /api/v1/orders | Get a list of orders based on role. | Yes | All |
| GET | /api/v1/orders/{id} | Get details for a single order. | Yes | All |
| PUT | /api/v1/orders/{id}/pickup | Pick up an order for preparation. | Yes | `WAREHOUSE` |
| PUT | /api/v1/orders/{id}/transit | Mark an order as in transit. | Yes | `WAREHOUSE` |
| PUT | /api/v1/orders/{id}/receive | Confirm receipt of an order. | Yes | `PRODUCTION_LINE` |
| PUT | /api/v1/orders/{id}/status | (Admin) Manually edit an order's status. | Yes | `ADMIN` |
| DELETE | /api/v1/orders/{id} | (Admin) Delete an order. | Yes | `ADMIN` |

## 4. Error Handling

- `400 Bad Request`: Invalid input data (e.g., missing fields, invalid format).
- `401 Unauthorized`: User is not authenticated.
- `403 Forbidden`: User is authenticated but does not have the required role for the action.
- `404 Not Found`: The requested resource (e.g., order) does not exist.
- `409 Conflict`: Invalid state transition (e.g., trying to pick up an order that is already in transit).
- `500 Internal Server Error`: Unexpected server-side error.
