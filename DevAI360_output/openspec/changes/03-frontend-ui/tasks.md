# Tasks: Shopfloor App Frontend Implementation

**Paths:** All frontend code under `frontend/`.

## Views/Pages (React)

- [ ] Create `src/pages/LoginPage.tsx` for the user authentication form.
- [ ] Create `src/pages/DashboardPage.tsx` which will act as the main container and router for role-specific dashboards.
- [ ] Implement client-side routing (e.g., using React Router) to handle navigation.

## Components

- [ ] Create a reusable `src/components/common/OrderCard.tsx` component.
- [ ] Create `src/components/production/CreateOrderForm.tsx`.
- [ ] Create `src/components/production/MyOrdersList.tsx`.
- [ ] Create `src/components/warehouse/NewOrdersQueue.tsx`.
- [ ] Create `src/components/warehouse/AssignedOrdersList.tsx`.
- [ ] Create `src/components/admin/AllOrdersTable.tsx` with filtering and searching logic.
- [ ] Create `src/components/admin/EditOrderModal.tsx`.
- [ ] Create shared components for buttons, inputs, and modals in `src/components/common/`.

## Services / State Management

- [ ] Create `src/services/api.ts` to encapsulate all `fetch` or `axios` calls to the backend, including setting the auth header.
- [ ] Implement a state management solution (e.g., Redux Toolkit or React Context) to manage user session, roles, and shared application state.
- [ ] Create an `auth` slice/context to handle user login, logout, and storing the JWT.
- [ ] Create an `orders` slice/context to manage fetching and updating order data across different components.

## Styling

- [ ] Set up a global stylesheet and a layout structure that ensures a responsive design for desktop, tablet, and mobile.
- [ ] Create CSS modules or use a framework (like Tailwind CSS) for component-level styling.
- [ ] Design clear visual cues for order statuses (e.g., color-coded badges).

## Integration

- [ ] Configure a proxy in the development server (e.g., in `vite.config.ts` or `webpack.config.js`) to forward `/api` requests to the backend server.
- [ ] Implement UI feedback for API states: loading spinners, success messages, and error notifications.
