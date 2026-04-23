# Detailed Design: Shopfloor Material Supply App

This document provides a consolidated summary of the detailed design for the Shopfloor Material Supply App, covering the data model, backend API, and frontend UI.

## 1. Data Model

The data model is designed around three core entities: `users`, `delivery_orders`, and `audit_logs`.

- **`users`**: Stores user information, including their role (`PRODUCTION`, `WAREHOUSE`, `ADMIN`).
- **`delivery_orders`**: Represents a material request with a status that tracks its lifecycle (`NEW`, `IN_PREPARATION`, `IN_TRANSIT`, `COMPLETED`).
- **`audit_logs`**: Provides an immutable record of all significant actions performed on an order, ensuring accountability.

For detailed schemas, see `01-data-model/proposal.md`.

## 2. Backend API

The backend will be a Go-based REST API secured with JWT authentication.

- **Authentication**: A `POST /api/auth/login` endpoint will issue JWTs. A middleware will protect all other endpoints.
- **Core Endpoints**: The API provides standard CRUD operations on `/api/orders` and `/api/users`.
- **Business Logic**: State transitions for orders are managed through a dedicated `PUT /api/orders/{id}/status` endpoint, with role-based rules enforced by the `OrderService`.
- **Admin Capabilities**: Admins have full access to manage users and can manually edit or delete orders, with all such actions being logged.

For a full list of endpoints and service descriptions, see `02-backend-api/proposal.md`.

## 3. Frontend UI

The frontend will be a responsive React application with role-based views.

- **Technology**: Built with React and TypeScript, using React Router for navigation and a lightweight state management solution.
- **User Experience**:
  - **Production users** will have a simple interface to create new orders and track their progress.
  - **Warehouse users** will have a dashboard to manage new and in-progress orders.
  - **Admins** will have a comprehensive view of all orders with powerful search and management tools.
- **Component-Based Design**: The UI will be built from a set of reusable components, ensuring consistency and maintainability.

For detailed view descriptions and component breakdowns, see `03-frontend-ui/proposal.md`.
```