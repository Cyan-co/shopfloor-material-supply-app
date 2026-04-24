# Phase B: Business Architecture (BA)

## 1. Business Domain

### Domain
The business domain is **Shopfloor Material Logistics**. This domain covers the processes, rules, and actors involved in requesting materials from a central warehouse and delivering them to a specific production line on a manufacturing shop floor.

### Process Description
The core business process is the "Delivery Order Fulfillment." This process begins when a user on the production line identifies a need for materials and creates a digital request (a "Delivery Order"). The request is then received and processed by warehouse staff. The process ends when the materials are physically delivered to the production line and the receipt is digitally confirmed. The entire process is monitored and managed by administrators who can intervene if necessary.

---

## 2. Business Actors & Roles

| Role | Responsibilities |
|---|---|
| **Production Line User** | - Creates new Delivery Orders for required materials. <br> - Views the status of their own active orders. <br> - Confirms the receipt of materials to complete an order. |
| **Warehouse User** | - Monitors for new Delivery Orders. <br> - "Picks up" a new order to begin processing, assigning it to themselves. <br> - Prepares the materials for delivery. <br> - Marks the order as "In Transit" when it leaves the warehouse. |
| **Admin** | - Has complete visibility over all Delivery Orders in the system, regardless of status. <br> - Can manually change the status of any order to correct errors. <br> - Can delete orders from the system. <br> - Manages user accounts and roles. |

---

## 3. Business Process Flow (The Golden Path)

The lifecycle of a Delivery Order follows a strict, linear progression. Each state change is triggered by a specific action from a designated business actor.

**`NEW`** -> **`IN_PREPARATION`** -> **`IN_TRANSIT`** -> **`COMPLETED`**

- **`NEW`**: A Production Line User creates a Delivery Order. This is the initial state.
- **`IN_PREPARATION`**: A Warehouse User "picks up" a `NEW` order.
- **`IN_TRANSIT`**: The assigned Warehouse User marks the `IN_PREPARATION` order as ready for delivery.
- **`COMPLETED`**: The original Production Line User confirms they have received the `IN_TRANSIT` order. This is a terminal state.

---

## 4. Business Rules

These rules are non-negotiable and must be enforced by the application's backend logic.

| Rule ID | Rule Description | Implementation Constraint |
|---|---|---|
| BR-001 | **State Transition Integrity** | A Delivery Order's status can only change in the sequence defined in the Business Process Flow. (e.g., An order cannot go from `NEW` to `IN_TRANSIT`). | The order processing service must contain a state machine that validates every status change request against the allowed transitions. Reject any invalid requests with a `400 Bad Request` error. |
| BR-002 | **Role-Based Action Control** | Actions that trigger state changes are restricted to specific roles. <br> - `NEW` -> `IN_PREPARATION`: Only a `Warehouse User` can trigger. <br> - `IN_PREPARATION` -> `IN_TRANSIT`: Only the *assigned* `Warehouse User` can trigger. <br> - `IN_TRANSIT` -> `COMPLETED`: Only the *originating* `Production Line User` can trigger. | The API endpoint for each state change action must be protected by RBAC middleware. The logic must also verify the specific user (e.g., `assigned_warehouse_user_id`) where applicable. |
| BR-003 | **Order Immutability** | Once a Delivery Order is in a terminal state (`COMPLETED`), its core data (Material Name, Quantity, etc.) cannot be edited. | The API must reject any `UPDATE` requests for orders where the `status` is `COMPLETED`. |
| BR-004 | **Admin Override Authority** | An `Admin` can bypass the standard process flow and change an order's status to any other valid state. | A dedicated admin-only API endpoint (`/admin/orders/{id}/status`) must exist that allows setting the status directly, bypassing the standard state machine constraints. |
| BR-005 | **Audit Trail Mandate** | Every action performed by an `Admin` (status change, deletion) MUST be accompanied by a reason, which is logged in the immutable audit trail. | The admin-only API endpoints for editing or deleting an order must require a `reason` field in the request body. This field must be saved to the `order_audit_log` table. |
| BR-006 | **Order Deletion is Logical** | When an `Admin` deletes an order, it must be a "soft delete." The record should be hidden from standard views but remain in the database for auditing purposes. | The `orders` table must have an `is_deleted` boolean flag. The standard "delete" API action should set this flag to `true` and record the event in the audit log. Standard list APIs should filter out records where `is_deleted = true`. |

---
