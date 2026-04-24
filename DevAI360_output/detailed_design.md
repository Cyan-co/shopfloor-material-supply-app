# Low-Level Design

## Database Schema

### Tables

-   **delivery_orders**
    -   `id` (BIGINT, Primary Key, Auto-increment)
    -   `material_name` (VARCHAR(255), Not Null)
    -   `quantity` (INT, Not Null)
    -   `status` (VARCHAR(50), Not Null, Default: 'NEW') - Values: NEW, PREPARING, IN_TRANSIT, COMPLETED
    -   `requester_id` (BIGINT, Foreign Key to `users.id`)
    -   `handler_id` (BIGINT, Foreign Key to `users.id`, Nullable)
    -   `created_at` (TIMESTAMP, Default: CURRENT_TIMESTAMP)
    -   `updated_at` (TIMESTAMP, Default: CURRENT_TIMESTAMP on update)

-   **users**
    -   `id` (BIGINT, Primary Key, Auto-increment)
    -   `username` (VARCHAR(255), Not Null, Unique)
    -   `password_hash` (VARCHAR(255), Not Null)
    -   `created_at` (TIMESTAMP, Default: CURRENT_TIMESTAMP)

-   **roles**
    -   `id` (INT, Primary Key, Auto-increment)
    -   `name` (VARCHAR(50), Not Null, Unique) - Values: ROLE_REQUESTER, ROLE_HANDLER, ROLE_ADMIN

-   **user_roles**
    -   `user_id` (BIGINT, Foreign Key to `users.id`)
    -   `role_id` (INT, Foreign Key to `roles.id`)
    -   Primary Key (`user_id`, `role_id`)

### Relationships

-   A `delivery_order` has one `requester` (a `user`).
-   A `delivery_order` can have one `handler` (a `user`).
-   A `user` can have multiple `roles`.

## Backend Design (Java 21)

### Package Structure

```
com.cyan.shopfloor
├── controller/
│   ├── OrderController.java
│   └── AuthController.java
├── service/
│   ├── OrderService.java
│   └── UserService.java
├── repository/
│   ├── OrderRepository.java
│   ├── UserRepository.java
│   └── RoleRepository.java
├── model/
│   ├── DeliveryOrder.java
│   ├── User.java
│   └── Role.java
├── dto/
│   ├── OrderRequest.java
│   └── UserLogin.java
└── security/
    └── WebSecurityConfig.java
```

### API Endpoints

-   **Authentication**
    -   `POST /api/auth/login` - Authenticate user, return JWT.

-   **Orders**
    -   `POST /api/orders` - Create a new delivery order (Requires: ROLE_REQUESTER).
    -   `GET /api/orders` - Get a list of all orders (Requires: any authenticated role).
    -   `GET /api/orders/{id}` - Get details for a specific order (Requires: any authenticated role).
    -   `PUT /api/orders/{id}/pickup` - Warehouse user picks up the order. Sets status to `PREPARING` (Requires: ROLE_HANDLER).
    -   `PUT /api/orders/{id}/transit` - Warehouse user marks the order as in transit. Sets status to `IN_TRANSIT` (Requires: ROLE_HANDLER).
    -   `PUT /api/orders/{id}/receive` - Production line user receives the order. Sets status to `COMPLETED` (Requires: ROLE_REQUESTER).
    -   `PUT /api/orders/{id}` - Manually update order details (Requires: ROLE_ADMIN).
    -   `DELETE /api/orders/{id}` - Delete an order (Requires: ROLE_ADMIN).

## Frontend Design (Angular 17)

### Module Structure

-   **CoreModule** (Guards, Interceptors)
-   **SharedModule** (Common components, pipes, directives)
-   **AuthModule** (Login page)
-   **OrdersModule** (Handles creation, viewing, and updating orders)
-   **AdminModule** (Admin dashboard for managing orders)

### Key Components

-   `LoginComponent` (AuthModule) - User login form.
-   `OrderListComponent` (OrdersModule) - Displays a real-time list of all orders, filterable by status.
-   `OrderDetailComponent` (OrdersModule) - Shows details of a single order and actions based on user role (e.g., "Receive" button).
-   `OrderCreateComponent` (OrdersModule) - A form for production line users to create new material requests.
-   `AdminDashboardComponent` (AdminModule) - A view for admins to see all orders with options to edit or delete them.

### Services

-   `AuthService` - Handles user authentication, token storage, and logout.
-   `OrderService` - Manages all CRUD operations for delivery orders.
-   `AdminService` - Provides admin-specific functionalities.
-   `AuthGuard` - Protects routes based on user authentication and roles.
