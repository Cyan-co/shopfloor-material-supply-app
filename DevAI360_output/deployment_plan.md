# Deployment Plan: Shopfloor Material Supply System

This document outlines the steps required to deploy the Shopfloor Material Supply System to a production-like environment.

## 1. Prerequisites

- **Application Server:** A server running a Linux distribution (e.g., Ubuntu 22.04) with Java 17 (or later) installed.
- **Database Server:** A running instance of PostgreSQL (Version 14 or later).
- **Web Server/Proxy:** A web server like Nginx to serve the frontend assets and proxy API requests.
- **Network Access:** The application server must be able to connect to the PostgreSQL database. User clients must be able to connect to the web server over HTTP/S.

## 2. Step 1: Database Setup

1.  **Create Database and User:**
    - A dedicated PostgreSQL database (e.g., `shopfloor_prod`) and a user with full privileges on this database must be created.
2.  **Run Migrations:**
    - The backend application uses Flyway to manage database schema migrations.
    - Upon its first launch, the Spring Boot application will automatically detect the database state and run all necessary migration scripts located in its resources (`db/migration`) to create the `shopfloor` schema and all required tables (`users`, `delivery_orders`, `audit_logs`).

## 3. Step 2: Backend Deployment (Spring Boot)

1.  **Build the Application:**
    - From the `backend/` directory, run the Maven command to produce the executable JAR file:
      ```bash
      mvn clean package
      ```
    - This will create a file like `shopfloor-0.0.1-SNAPSHOT.jar` in the `backend/target/` directory.
2.  **Configure:**
    - Create an `application-prod.properties` file on the application server.
    - This file must contain the production database URL, username, password, and the JWT secret key.
      ```properties
      spring.datasource.url=jdbc:postgresql://<db_host>:5432/shopfloor_prod
      spring.datasource.username=<db_user>
      spring.datasource.password=<db_password>
      jwt.secret=<your_strong_jwt_secret>
      ```
3.  **Deploy and Run:**
    - Copy the executable JAR to the application server.
    - Run the application as a service (e.g., using `systemd`). The command to run the application is:
      ```bash
      java -jar shopfloor-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
      ```

## 4. Step 3: Frontend Deployment (Angular)

1.  **Build the Application:**
    - From the `frontend/` directory, run the Angular CLI command to build the static assets for production:
      ```bash
      ng build --configuration production
      ```
    - This will create a `dist/` directory containing all the necessary HTML, CSS, and JavaScript files.
2.  **Deploy to Web Server:**
    - Copy the contents of the `frontend/dist/shopfloor-app/` directory to the web root of the Nginx server (e.g., `/var/www/html`).
3.  **Configure Nginx:**
    - Configure Nginx to serve the Angular application.
    - Set up a reverse proxy rule to forward all requests for `/api/*` to the backend Spring Boot application running on its port (e.g., `http://localhost:8080`). This avoids CORS issues.
    - Ensure a "catch-all" rule is in place to redirect all other routes to `index.html` to support the Angular router.

## 5. Verification

1.  **Backend Health Check:**
    - Access the backend's health endpoint (e.g., `http://<app_server_ip>:8080/actuator/health`). It should return `{"status":"UP"}`.
2.  **Frontend Access:**
    - Open a web browser and navigate to the application's URL (e.g., `http://<your_domain>`). The login page should load correctly.
3.  **End-to-End Test:**
    - Log in with a test Production Operator account.
    - Create a new material request.
    - Log out and log in with a Warehouse Staff account.
    - Verify the new request appears in the queue.
    - Accept and process the order through its lifecycle.
    - Verify all steps complete successfully.
