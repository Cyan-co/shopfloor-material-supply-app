# Tasks: Shopfloor Material Supply System Backend API Implementation

**Paths:** All backend code under `backend/src/main/java/com/example/shopfloor/`.

---

## Controller Layer

- [ ] **Task 1: Create `DeliveryOrderController`**
  - File: `backend/src/main/java/com/example/shopfloor/controller/DeliveryOrderController.java`
  - Annotate with `@RestController`, `@RequestMapping("/api/orders")`.
  - Implement all endpoints defined in `proposal.md`, delegating logic to `DeliveryOrderService`.
  - Use `@AuthenticationPrincipal` to get the current user for service method calls.

- [ ] **Task 2: Create `AuthController`**
  - File: `backend/src/main/java/com/example/shopfloor/controller/AuthController.java`
  - Implement a `POST /api/auth/login` endpoint that takes credentials and returns a JWT.

---

## DTOs

- [ ] **Task 3: Create DTOs for `DeliveryOrder`**
  - Path: `backend/src/main/java/com/example/shopfloor/dto/`
  - Create `CreateDeliveryOrderDto.java` with JSR-380 validation.
  - Create `UpdateOrderStatusDto.java`.
  - Create `DeleteOrderDto.java`.
  - Create `DeliveryOrderDto.java` for responses.

- [ ] **Task 4: Create DTOs for Auditing and Auth**
  - Create `AuditLogDto.java`.
  - Create `LoginRequestDto.java` and `JwtResponseDto.java`.

---

## Service Layer

- [ ] **Task 5: Implement `DeliveryOrderService.createOrder()`**
  - Apply `@PreAuthorize("hasRole('PRODUCTION_OPERATOR')")`.
  - Validate input, set initial status to `PENDING`, set requester, and save.

- [ ] **Task 6: Implement `DeliveryOrderService.getOrders()`**
  - Implement role-based filtering logic:
    - `PRODUCTION_OPERATOR`: Sees their own created orders.
    - `WAREHOUSE_STAFF`: Sees `PENDING` and `ACCEPTED` orders.
    - `ADMINISTRATOR`: Sees all orders.

- [ ] **Task 7: Implement `DeliveryOrderService.updateOrderStatus()`**
  - Load the order.
  - Check if the state transition is valid based on the state machine.
  - Check if the user's role permits the transition.
  - If the user is an admin, log the action to the audit trail.
  - Update status and save.

- [ ] **Task 8: Implement `DeliveryOrderService.deleteOrder()`**
  - Apply `@PreAuthorize("hasRole('ADMINISTRATOR')")`.
  - Perform a soft delete.
  - Log the action to the audit trail via `AuditLogService`.

- [ ] **Task 9: Implement `AuditLogService`**
  - Create a single public method to create and save new `AuditLog` entities.

---

## Security

- [ ] **Task 10: Configure `SecurityConfig`**
  - File: `backend/src/main/java/com/example/shopfloor/config/SecurityConfig.java`
  - Enable method security with `@EnableMethodSecurity`.
  - Configure the `SecurityFilterChain` to be stateless, disable CSRF, and set up authorization rules (`/api/auth/**` is public, `/api/**` is authenticated).

- [ ] **Task 11: Implement JWT Utilities**
  - Create a `JwtUtil` class to generate, parse, and validate JWTs.
  - Create a `JwtAuthFilter` to run on each request, validate the token, and set the `Authentication` in the `SecurityContextHolder`.

- [ ] **Task 12: Implement `UserDetailsService`**
  - Create `AppUserDetailsService` that loads a `User` entity from the database and converts it to Spring Security's `UserDetails`.

---

## Error Handling

- [ ] **Task 13: Create Custom Exceptions**
  - Path: `backend/src/main/java/com/example/shopfloor/exception/`
  - Create `ResourceNotFoundException.java`.
  - Create `InvalidStatusTransitionException.java`.

- [ ] **Task 14: Create `GlobalExceptionHandler`**
  - Annotate with `@RestControllerAdvice`.
  - Create `@ExceptionHandler` methods for custom exceptions, `AccessDeniedException`, and `MethodArgumentNotValidException` to return consistent JSON error responses.
