# Frontend UI Proposal

This document outlines the design for the Frontend UI of the Shopfloor Material Supply App. The UI will be a responsive web application built with React and TypeScript.

## 1. Views and Components

### Common Components
- **`LoginPage.tsx`**: A simple form for user login.
- **`Navbar.tsx`**: Top navigation bar with logout button and user info.
- **`OrderCard.tsx`**: A reusable component to display a summary of a delivery order.
- **`OrderList.tsx`**: A component to display a list of `OrderCard` components.
- **`CreateOrderForm.tsx`**: A modal or form for creating a new order.

### Role-Based Views

- **Production User Dashboard**
  - Displays a list of orders created by the user.
  - A prominent "Create New Order" button that opens the `CreateOrderForm`.
  - An action button on each "In Transit" order to "Receive" it.

- **Warehouse User Dashboard**
  - Displays two lists of orders:
    1.  "New Orders" - All orders with `NEW` status.
    2.  "My Orders" - Orders assigned to the current user (`IN_PREPARATION`, `IN_TRANSIT`).
  - Action buttons on orders to "Pick Up" (for new orders) and "Mark as In Transit" (for in-preparation orders).

- **Admin Dashboard**
  - Displays a comprehensive list of all orders, regardless of status.
  - Includes search and filter controls (by status, date, etc.).
  - Action buttons on each order to "Edit Status" or "Delete".

## 2. State Management

- **Strategy**: React Context API or a lightweight state management library like Zustand.
- **User State**: The logged-in user's information (including role and JWT) will be stored in a global state and persisted in local storage.
- **Data Fetching**: A dedicated API service (`api.ts`) will handle all communication with the backend, attaching the auth token to every request. React Query will be used for data fetching, caching, and state synchronization.

## 3. Routing

- **`/login`**: The login page.
- **`/`**: The main dashboard, which will render the correct view based on the user's role.
- **`/admin`**: The admin dashboard.

A protected route component will wrap the main routes to redirect unauthenticated users to the login page.
```