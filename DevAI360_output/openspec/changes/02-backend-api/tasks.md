# Tasks: Shopfloor Material Supply App Backend API Implementation

**Paths:** All backend code under `backend/`.

## Service Layer

- [ ] Implement `AuthService` with JWT generation and validation logic.
- [ ] Implement `OrderService` containing all business logic for order state transitions.
- [ ] Implement `AuditService` to handle the creation of log entries for all state-changing actions.
- [ ] Ensure all `OrderService` methods that modify state also call `AuditService`.
- [ ] Add validation logic to service methods (e.g., check for required fields, valid state transitions).

## Controller/Handler Layer (Go)

- [ ] Create handlers in `internal/api/handlers/` for authentication (`auth_handler.go`).
- [ ] Create handlers for order management (`order_handler.go`).
- [ ] Define request and response structs (DTOs) for all API endpoints in `internal/api/dto/`.
- [ ] Set up routing (e.g., using `chi` or `gin`) in `internal/api/router.go` to map endpoints to handlers.

## Security

- [ ] Implement middleware for JWT authentication to protect endpoints.
- [ ] Implement middleware for authorization (RBAC) that checks user roles against endpoint requirements.
- [ ] Implement password hashing (e.g., bcrypt) for user password storage.

## Error Handling

- [ ] Create custom error types for business logic failures (e.g., `ErrInvalidStateTransition`).
- [ ] Implement a global error handling middleware to convert service layer errors into appropriate HTTP status codes and JSON responses.
