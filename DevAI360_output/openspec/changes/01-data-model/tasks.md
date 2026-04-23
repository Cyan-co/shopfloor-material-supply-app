# Data Model Implementation Tasks

This document lists the tasks required to implement the data model for the Shopfloor Material Supply App.

- [ ] **Task 1: Set Up Database and Migrations**
  - **Description**: Initialize the PostgreSQL database and configure a migration tool (e.g., `golang-migrate/migrate`).
  - **Acceptance Criteria**: The database is running and accessible from the backend application. The migration tool is set up and can connect to the database.

- [ ] **Task 2: Create `users` Table Migration**
  - **Description**: Write the SQL migration script to create the `users` table as defined in the proposal.
  - **Acceptance Criteria**: The `up` migration creates the table with the correct columns, constraints, and indexes. The `down` migration correctly drops the table.

- [ ] **Task 3: Create `delivery_orders` Table Migration**
  - **Description**: Write the SQL migration script to create the `delivery_orders` table.
  - **Acceptance Criteria**: The migration includes foreign key constraints to the `users` table for `requestor_id` and `processor_id`.

- [ ] **Task 4: Create `audit_logs` Table Migration**
  - **Description**: Write the SQL migration script to create the `audit_logs` table.
  - **Acceptance Criteria**: The migration includes foreign key constraints to `delivery_orders` and `users`.

- [ ] **Task 5: Implement Go Data Models**
  - **Description**: Create the Go structs corresponding to the `users`, `delivery_orders`, and `audit_logs` tables.
  - **Acceptance Criteria**: The structs are defined in a `models` package with correct data types and struct tags for JSON and database mapping.

- [ ] **Task 6: Implement Enums**
  - **Description**: Define the `UserRole` and `OrderStatus` enums in Go.
  - **Acceptance Criteria**: The enums are implemented as custom types with constants for each possible value.
```