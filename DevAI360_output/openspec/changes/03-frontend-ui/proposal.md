# Proposal: Shopfloor Material Supply App Frontend UI

**Ref:** Depends on `@openspec/changes/01-data-model`, `@openspec/changes/02-backend-api`

## 1. View Architecture

### LoginView
- **Purpose**: Provides a form for users to authenticate.
- **User Roles**: All roles.
- **Layout**: A simple centered form with fields for email and password.

### DashboardView (Role-based)
- **Purpose**: The main application view that dynamically renders content based on the logged-in user's role.
- **User Roles**: All roles.
- **Layout**: Contains a main navigation/header and a content area. The content area will display one of the role-specific dashboards below.

#### 1a. ProductionDashboard
- **Purpose**: Allows Production Line Users to create new requests and track their existing ones.
- **User Roles**: `PRODUCTION`
- **Layout**: A two-section layout. One section contains the `CreateOrderForm` component. The other section contains the `MyOrdersList` component.

#### 1b. WarehouseDashboard
- **Purpose**: Enables Warehouse Users to view and process new and assigned material requests.
- **User Roles**: `WAREHOUSE`
- **Layout**: A two-section layout. One section displays the `NewOrdersQueue` component. The other displays the `AssignedOrdersList` component.

#### 1c. AdminDashboard
- **Purpose**: Gives Admins a complete overview of all orders with management capabilities.
- **User Roles**: `ADMIN`
- **Layout**: A full-page layout dominated by the `AllOrdersTable` component, with filter and search controls at the top.

## 2. Component Requirements

| Component | Purpose | Data Source |
|-----------|---------|-------------|
| `OrderCard` | A reusable component to display a summary of a single order. Shows status, material, and quantity. Contains action buttons. | Prop-driven. |
| `CreateOrderForm` | A form with inputs for `material_name`, `quantity`, and `delivery_location`. | `POST /api/orders` |
| `MyOrdersList` | Displays a list of orders created by the current Production User. | `GET /api/orders?requestor=me` |
| `NewOrdersQueue` | Displays a list of all orders with `NEW` status for warehouse staff. | `GET /api/orders?status=NEW` |
| `AssignedOrdersList`| Displays orders assigned to the current Warehouse User. | `GET /api/orders?processor=me` |
| `AllOrdersTable` | A comprehensive, filterable, and searchable table of all orders for admins. | `GET /api/orders/all` |
| `EditOrderModal` | A modal form for admins to change an order's status and provide a reason. | `PUT /api/orders/{id}/status` |

## 3. User Interactions

### Create Order
- **Trigger**: Production User clicks "Submit" on the `CreateOrderForm`.
- **Action**: A POST request is sent to `/api/orders`.
- **Feedback**: The form is cleared, a success notification appears, and the new order appears in the `MyOrdersList`.

### Pick Up Order
- **Trigger**: Warehouse User clicks the "Pick Up" button on an order in the `NewOrdersQueue`.
- **Action**: A PUT request is sent to `/api/orders/{id}/pickup`.
- **Feedback**: The order is removed from the "New" list and appears in the user's "Assigned Orders" list.

### Receive Order
- **Trigger**: Production User clicks the "Receive" button on an "In Transit" order.
- **Action**: A PUT request is sent to `/api/orders/{id}/receive`.
- **Feedback**: The order's status badge changes to "Completed" and the button disappears.

## 4. Role-based Visibility

| Element | `PRODUCTION` | `WAREHOUSE` | `ADMIN` |
|---------|----------|----------|---------|
| Create Order Form | ✓ | ✗ | ✗ |
| "Pick Up" Button | ✗ | ✓ (on `NEW` orders) | ✗ |
| "In Transit" Button | ✗ | ✓ (on their `IN_PREP` orders) | ✗ |
| "Receive" Button | ✓ (on their `IN_TRANSIT` orders) | ✗ | ✗ |
| "Edit" / "Delete" Buttons | ✗ | ✗ | ✓ |
| Admin Dashboard | ✗ | ✗ | ✓ |

## 5. API Integration

| Component | Endpoint | Method | Purpose |
|-----------|----------|--------|---------|
| `LoginView` | `/api/auth/login` | POST | Authenticate user and store JWT. |
| `CreateOrderForm` | `/api/orders` | POST | Create a new delivery order. |
| `NewOrdersQueue` | `/api/orders` | GET | Fetch all orders with `NEW` status. |
| `AllOrdersTable` | `/api/orders/all` | GET | Fetch all orders for the admin view. |
| `OrderCard` | `/api/orders/{id}/*` | PUT | Update order status via action buttons. |
