# Deployment Plan: Shopfloor Material Supply App

## 1. Prerequisites

- A provisioned PostgreSQL database with connection details (host, port, user, password, database name).
- A container orchestration environment (e.g., Kubernetes, Docker Swarm) for deploying the backend and frontend services.
- CI/CD pipelines configured for building, testing, and deploying Go and React applications.
- Environment variables configured for the backend service (database connection string, JWT secret key).

## 2. Step 1: Database Setup

- **Action**: Apply database migration scripts.
- **Details**: The Go backend service will use a library like `golang-migrate/migrate`. On startup, the service will automatically apply any new migration scripts located in the `/migrations` directory to the target database. The initial migration will create the `users`, `delivery_orders`, and `audit_logs` tables.

## 3. Step 2: Backend Deployment (Go)

- **Build**: The CI pipeline will build a statically-linked Go binary inside a minimal Docker container (e.g., `FROM scratch`).
- **Push**: The resulting Docker image will be pushed to a container registry.
- **Deploy**: The deployment process will pull the new image and perform a rolling update of the backend service in the target environment.

## 4. Step 3: Frontend Deployment (React)

- **Build**: The CI pipeline will build the static React assets (`npm run build`).
- **Containerize**: The static assets will be placed into a lightweight web server container (e.g., Nginx).
- **Push**: The resulting Docker image will be pushed to a container registry.
- **Deploy**: The deployment process will perform a rolling update of the frontend service. The Nginx server will be configured to serve the React application and proxy all `/api/*` requests to the backend service.

## 5. Verification

- **Health Checks**: Both backend and frontend services must expose a health check endpoint (e.g., `/healthz`). The orchestration platform should use this to verify service health.
- **Smoke Test**:
    1. Manually access the frontend application's URL and verify the login page loads.
    2. Log in with a test "Production" user.
    3. Create a new delivery order.
    4. Verify the order appears in the user's order list with the status "New".
    5. Log in with a test "Admin" user and verify the new order is visible on the main dashboard.
