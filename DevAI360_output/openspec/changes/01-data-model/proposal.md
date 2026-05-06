# Proposal: Shopfloor Material Supply System Data Model

Based on the requirement and available architecture context, the following data model has been designed to support the full business process.

## 1. Entities Overview

| Entity | Description | Key Responsibility |
|---|---|---|
| **User** | Represents an employee who can log in and interact with the system based on their role. | Manages identity, authentication, and authorization (role). |
| **DeliveryOrder** | The core entity representing a single material supply request from the shop floor. | Tracks the entire lifecycle of a material request from creation to completion. |
| **AuditLog** | Records all administrative actions for compliance and traceability. | Provides an immutable history of manual interventions (status changes, deletions). |

## 2. Schema: `shopfloor`

> All tables reside in the `shopfloor` schema. Primary keys use UUID. Timestamps are stored as TIMESTAMPTZ (UTC).

### Table: `users`

- **Purpose:** Stores user accounts and their assigned roles.
- **Columns:**

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK, NOT NULL | Auto-generated unique identifier. |
| `username` | VARCHAR(100) | NOT NULL, UNIQUE | The unique login name for the user. |
| `password_hash` | VARCHAR(255) | NOT NULL | The securely hashed password. |
| `role` | VARCHAR(50) | NOT NULL, CHECK (role IN ('PRODUCTION_OPERATOR', 'WAREHOUSE_STAFF', 'ADMINISTRATOR')) | User's role, controlling permissions. |
| `created_at` | TIMESTAMPTZ | NOT NULL, DEFAULT now() | Record creation timestamp. |
| `updated_at` | TIMESTAMPTZ | NOT NULL | Last modification timestamp. |

### Table: `delivery_orders`

- **Purpose:** Tracks each material supply request and its state.
- **Columns:**

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK, NOT NULL | Auto-generated unique identifier. |
| `material_name` | VARCHAR(255) | NOT NULL | Name/code of the requested material. |
| `quantity` | INT | NOT NULL, CHECK (quantity > 0) | The amount of material requested. |
| `unit_of_measure`| VARCHAR(50) | NOT NULL | The unit for the quantity (e.g., 'pcs', 'kg'). |
| `target_production_line` | VARCHAR(100)| NOT NULL | The production line requesting the material. |
| `notes` | TEXT | | Optional free-text notes from the requester. |
| `status` | VARCHAR(50) | NOT NULL, CHECK (status IN ('PENDING', 'ACCEPTED', 'IN_TRANSIT', 'DELIVERED', 'CANCELLED')) | Current stage of the order in the workflow. |
| `priority` | VARCHAR(50) | NOT NULL, DEFAULT 'NORMAL', CHECK (priority IN ('NORMAL', 'URGENT')) | The priority level of the request. |
| `requester_id` | UUID | FK -> users.id, NOT NULL | The user who created the request. |
| `handler_id` | UUID | FK -> users.id | The warehouse user who is processing the order. |
| `is_deleted` | BOOLEAN | NOT NULL, DEFAULT FALSE | Flag for soft-deleting orders. |
| `created_at` | TIMESTAMPTZ | NOT NULL, DEFAULT now() | Record creation timestamp. |
| `updated_at` | TIMESTAMPTZ | NOT NULL | Last modification timestamp. |

### Table: `audit_logs`

- **Purpose:** Immutable log of all administrative changes; rows are never updated or deleted.
- **Columns:**

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK, NOT NULL | Auto-generated unique identifier. |
| `order_id` | UUID | FK -> delivery_orders.id, NOT NULL | The order that was acted upon. |
| `user_id` | UUID | FK -> users.id, NOT NULL | The administrator who performed the action. |
| `action` | VARCHAR(50) | NOT NULL | Action type (e.g., 'STATUS_CHANGE', 'DELETE'). |
| `details` | JSONB | | Additional context, like the reason for change. |
| `created_at` | TIMESTAMPTZ | NOT NULL, DEFAULT now() | When the action occurred. |

## 3. Relationships

- **User -> DeliveryOrder:** One-to-Many. One User (requester) can create many DeliveryOrders. One User (handler) can process many DeliveryOrders.
- **DeliveryOrder -> User:** Many-to-One. Each DeliveryOrder is associated with exactly one `requester_id` and optionally one `handler_id`.
- **AuditLog -> DeliveryOrder & User:** Many-to-One. Each audit log entry is linked to one specific order and the one administrator who made the change.

## 4. Status / Lifecycle

The `delivery_orders.status` field follows this lifecycle. Transitions are enforced at the service layer:

| Status | Meaning | Allowed Next States |
|---|---|---|
| `PENDING` | Order created by a Production Operator, awaiting pickup. | `ACCEPTED`, `CANCELLED` |
| `ACCEPTED` | A Warehouse Staff member has accepted and is preparing the order. | `IN_TRANSIT`, `CANCELLED` |
| `IN_TRANSIT`| The order is on its way to the production line. | `DELIVERED`, `CANCELLED` |
| `DELIVERED` | The order has been received at the production line. Terminal state. | (none) |
| `CANCELLED` | The order was cancelled by an administrator. Terminal state. | (none) |

## 5. Indexes & Performance Considerations

| Table | Index Column(s) | Type | Reason |
|---|---|---|---|
| `users` | `username` | B-TREE (UNIQUE) | Fast user lookup during login. |
| `delivery_orders` | `status` | B-TREE | Fast filtering of orders by status for queues. |
| `delivery_orders` | `requester_id` | B-TREE | Efficiently query orders for a specific user. |
| `audit_logs` | `order_id` | B-TREE | Quickly retrieve the history for a specific order. |
| `audit_logs` | `created_at` | B-TREE | For time-range audit queries. |

This design directly supports the business process flow, enforces referential integrity, and provides all data required by the backend services and frontend views.
