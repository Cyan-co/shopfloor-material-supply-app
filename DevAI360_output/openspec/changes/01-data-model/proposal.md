# Proposal: Shopfloor Material Supply App Data Model

## 1. Entities Overview

- **User**: Represents any individual who logs into the system, including production staff, warehouse staff, and administrators.
- **DeliveryOrder**: The core entity representing a single material supply request from the shopfloor to the warehouse.
- **AuditLog**: An immutable log to track all status changes and administrative actions performed on a DeliveryOrder, ensuring accountability.

## 2. User Schema

- **Table Name**: `users`
- **Description**: Stores user account information and their assigned role.
- **Fields**:
    - `UUID id`: Primary key.
    - `String name`: User's full name.
    - `String email`: User's login email (must be unique).
    - `String password_hash`: Hashed password for security.
    - `UserRole role`: The user's role in the system (Enum: `PRODUCTION`, `WAREHOUSE`, `ADMIN`).
    - `DateTime created_at`: Timestamp of user creation.

## 3. DeliveryOrder Schema

- **Table Name**: `delivery_orders`
- **Description**: Represents a material request and its lifecycle.
- **Fields**:
    - `UUID id`: Primary key.
    - `String material_name`: Name of the material being requested.
    - `Integer quantity`: The quantity of the material needed.
    - `String delivery_location`: The specific production line or area where the material is needed.
    - `OrderStatus status`: The current status of the order (Enum).
    - `UUID requestor_id`: Foreign key to the `users` table (the user who created the order).
    - `UUID processor_id`: Foreign key to the `users` table (the warehouse user handling the order, nullable).
    - `DateTime created_at`: Timestamp of order creation.
    - `DateTime updated_at`: Timestamp of the last status update.

## 4. AuditLog Schema

- **Table Name**: `audit_logs`
- **Description**: Records every significant event in an order's lifecycle for tracking and auditing purposes.
- **Fields**:
    - `UUID id`: Primary key.
    - `UUID order_id`: Foreign key to the `delivery_orders` table.
    - `UUID changed_by_id`: Foreign key to the `users` table (who made the change).
    - `String reason`: A text field for admins to explain manual edits or deletions.
    - `OrderStatus previous_status`: The status of the order before the change.
    - `OrderStatus new_status`: The status of the order after the change.
    - `DateTime created_at`: Timestamp of the audit event.


## 5. Relationships

- A `User` can have many `DeliveryOrder`s (as a requestor).
- A `User` can process many `DeliveryOrder`s (as a processor).
- A `DeliveryOrder` belongs to one `requestor` (User) and can be assigned to one `processor` (User).
- A `DeliveryOrder` can have many `AuditLog` entries.

## 6. Status/Lifecycle (OrderStatus Enum)

- `NEW`: The initial state of an order after being created by a Production Line User.
- `IN_PREPARATION`: The state after a Warehouse User has accepted the order and is preparing the materials.
- `IN_TRANSIT`: The state when the materials have left the warehouse and are on their way to the production line.
- `COMPLETED`: The final state after the Production Line User has confirmed receipt of the materials.
