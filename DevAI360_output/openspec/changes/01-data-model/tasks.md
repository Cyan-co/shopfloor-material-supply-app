# Tasks: Shopfloor App Data Model Implementation

**Paths:** Backend code under `backend/`, Frontend code under `frontend/`.

## Backend (Go)

- [ ] Create `internal/models/user.go` to define the User struct and `UserRole` enum.
- [ ] Create `internal/models/delivery_order.go` to define the DeliveryOrder struct and `OrderStatus` enum.
- [ ] Create `internal/models/audit_log.go` to define the AuditLog struct.
- [ ] Implement database migration scripts (e.g., using `golang-migrate/migrate`) to create the `users`, `delivery_orders`, and `audit_logs` tables in PostgreSQL.
- [ ] Define repository interfaces in `internal/database/` for data access functions (e.g., `CreateUser`, `GetOrderByID`, `UpdateOrderStatus`).

## Frontend (React/TypeScript)

- [ ] Create `src/types/user.ts` to define the User interface and UserRole enum.
- [ ] Create `src/types/deliveryOrder.ts` to define the DeliveryOrder interface and OrderStatus enum.
