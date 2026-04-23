# Phase B: Business Architecture (BA)

## 1. Business Domain

### Domain
This architecture concerns the **Shopfloor Material Supply Chain**. It defines the process of ordering, preparing, and delivering materials from a warehouse to a production line within a manufacturing environment.

### Process Description
The core business process is the "Delivery Order" lifecycle. This process begins when a Production Line User requests materials and ends when those materials are confirmed as received. The system provides a digital, traceable, and auditable record of this entire workflow, managed through a series of defined states and transitions.

---

## 2. Business Actors & Roles

| Role | Responsibilities |
|---|---|
| **Production Line User** | - Creates new `Delivery Orders`. <br> - Views the status of their own orders. <br> - Confirms receipt of delivered orders, transitioning them to `COMPLETED`. |
| **Warehouse User** | - Views a queue of `NEW` Delivery Orders. <br> - "Picks up" a `NEW` order, which assigns it to them and moves it to `IN_PREPARATION`. <br> - Marks a prepared order as `IN_TRANSIT` for delivery. |
| **Admin** | - Has complete read-access to all orders in any state. <br> - Can manually change the state of any order for exception handling. <br> - Can delete any order. <br> - Manages user accounts and roles (Implied for future versions). |

---

## 3. Business Process Flow (The Golden Path)

The lifecycle of a Delivery Order follows a strict, linear state machine progression.

**NEW** -> **IN_PREPARATION** -> **IN_TRANSIT** -> **COMPLETED**

- **`NEW`**: The initial state of an order upon creation. It is unassigned and awaits action from the warehouse.
- **`IN_PREPARATION`**: The state after a Warehouse User has accepted the order. The materials are being gathered.
- **`IN_TRANSIT`**: The state indicating the materials have left the warehouse and are on their way to the production line.
- **`COMPLETED`**: The terminal state indicating the Production Line User has successfully received the materials.

An additional terminal state, **`DELETED`**, exists outside the golden path and is only accessible via an Admin action.

---

## 4. Business Rules

These rules are non-negotiable and must be enforced by the application's backend logic.

### State Transition Rules
1.  A Delivery Order can only be created in the `NEW` state.
2.  Transition from `NEW` to `IN_PREPARATION` is the only allowed path from `NEW`.
3.  Transition from `IN_PREPARATION` to `IN_TRANSIT` is the only allowed path from `IN_PREPARATION`.
4.  Transition from `IN_TRANSIT` to `COMPLETED` is the only allowed path from `IN_TRANSIT`.
5.  `COMPLETED` and `DELETED` are terminal states; no transitions are allowed from them.

### Role-Based Access Control (RBAC) Rules
1.  **Creation**: Only a `Production Line User` can create a `NEW` order.
2.  **Pick Up**: Only a `Warehouse User` can transition an order from `NEW` to `IN_PREPARATION`. The user performing this action is permanently assigned to the order.
3.  **Mark for Transit**: Only the assigned `Warehouse User` can transition their order from `IN_PREPARATION` to `IN_TRANSIT`.
4.  **Receive**: Only the originating `Production Line User` can transition an order from `IN_TRANSIT` to `COMPLETED`.
5.  **Admin Override**: An `Admin` can transition any order to any state (except from a terminal state).
6.  **Deletion**: Only an `Admin` can move an order to the `DELETED` state. A reason for this action must be recorded in the audit log.

### Immutability Rules
1.  The `Requestor Name` and original `Creation Timestamp` of an order are immutable and can never be changed.
2.  Once an order is in a terminal state (`COMPLETED` or `DELETED`), all its attributes are immutable.

### Exception Handling Rules
1.  Any manual state change performed by an `Admin` must require a "reason for change" text input, which must be stored in the immutable audit log.
