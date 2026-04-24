# Product Requirements Document (PRD): Shopfloor Material Supply App

## 1. Executive Summary
This document outlines the requirements for a web application designed to streamline the material supply process between the production line and the warehouse. The primary problem is the lack of a centralized, trackable system for material requests, leading to production delays and zero visibility into process bottlenecks. This application will provide a clear, real-time status view of all material requests, improving efficiency and enabling data-driven process improvements.

## 2. Goals and Success Criteria
- **Goal 1:** Increase the efficiency of the material supply process.
  - **Success Metric:** Reduce the average time from order creation to completion by 25% within 3 months of launch.
- **Goal 2:** Provide full visibility into the material request lifecycle.
  - **Success Metric:** All material requests are tracked in the system, with 100% of status changes logged and visible to relevant users.
- **Goal 3:** Establish a foundation for future process optimization.
  - **Success Metric:** The system successfully captures timestamp data for all status changes, allowing for bottleneck analysis.

## 3. Solution Recommendation
### Option 1: Phased MVP
- **Description:** A core web application focusing only on the essential workflow: create, prepare, transit, complete. Admin functions for editing/canceling are included. Defer insights like timestamp analysis and API integrations.
- **Pros:** Fastest time to market, lowest initial cost.
- **Cons:** Lacks scalability and data insights, may require significant refactoring for future integrations.

### Option 2: Strategic Platform Build
- **Description:** Build the application as a scalable platform from the start. This includes the core workflow, plus an API-first design and detailed timestamp logging for analytics.
- **Pros:** Future-proof, provides valuable process data immediately, easier to integrate with other systems.
- **Cons:** Higher initial development effort and cost.

### **Recommended Option: Strategic Platform Build**
- **Rationale:** The core business problem is not just moving materials, but understanding and optimizing the process. Building the platform with analytics and integration capabilities from the outset provides far greater long-term value. The slightly longer development time is a worthwhile investment to gain immediate insights into process bottlenecks.

## 4. User Roles and Personas
- **Production Line User:** Creates material supply requests for their designated production line. Needs a simple, fast interface to minimize time away from their primary tasks.
- **Warehouse User:** Views, picks up, and fulfills material requests. Needs a clear, prioritized list of open orders and the ability to update statuses quickly.
- **Admin:** Monitors the entire system. Has override capabilities to fix errors, cancel requests, and manage the system.

## 5. Functional Requirements
| ID      | Requirement                                                                                                  | Acceptance Criteria                                                                                                                                                                                                                         |
|---------|--------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **FR-001**  | A **Production Line User** shall be able to add one or more material requests to a Supply Cart and submit them as a single Delivery Order. | - User can search for materials by SKU.<br>- User can add multiple items to a cart.<br>- User can specify quantity for each item.<br>- User can submit the cart, which creates a single Delivery Order with a unique ID. |
| **FR-002**  | A **Warehouse User** shall be able to view a list of pending Delivery Orders.                                  | - The list displays all orders with the status "Open".<br>- The list is sorted by creation time (oldest first).<br>- Each item shows the Order ID, creation time, and requesting production line.                 |
| **FR-003**  | A **Warehouse User** shall be able to change the status of a Delivery Order to **"Preparing"**.                 | - The user can select an "Open" order and click an "Accept" button.<br>- The order status changes to "Preparing".<br>- The order is assigned to the accepting warehouse user.                                       |
| **FR-004**  | A **Warehouse User** shall be able to change the status of a Delivery Order to **"In Transit"**.                | - The user can select a "Preparing" order and click a "Dispatch" button.<br>- The order status changes to "In Transit".                                                                                             |
| **FR-005**  | A **Production Line User** shall be able to change the status of a Delivery Order to **"Completed"**.           | - The user can view their "In Transit" orders.<br>- The user can click a "Receive" button to change the status to "Completed".                                                                                          |
| **FR-006**  | An **Admin** shall be able to view and monitor all Delivery Orders regardless of status.                      | - The Admin dashboard displays a master list of all orders.<br>- The list can be filtered by status, user, and date range.                                                                                                   |
| **FR-007**  | An **Admin** shall be able to manually edit the status of any Delivery Order.                                 | - The Admin can select any order and choose a new status from a dropdown of all possible statuses.                                                                                                                         |
| **FR-008**  | An **Admin** shall be able to delete any Delivery Order.                                                        | - The Admin can select any order and click a "Delete" button.<br>- A confirmation prompt is displayed before deletion.                                                                                                      |
| **FR-009**  | The system shall enforce user roles (Production, Warehouse, Admin) to control access to functionality.        | - A user logged in as "Production" can only access production user functions.<br>- A user logged in as "Warehouse" can only access warehouse functions.<br>- An Admin has access to all functions.                 |
| **FR-010**  | The system shall log a timestamp for every status change of a Delivery Order.                                | - The database record for an order contains timestamp fields for "created", "prepared", "dispatched", and "completed".<br>- These timestamps are recorded automatically upon status change.                    |
| **FR-011**  | An **Admin** shall be able to change the status of any Delivery Order to **"Cancelled"**.                         | - "Cancelled" is available as a status in the Admin's edit view.<br>- A cancelled order is removed from active queues but remains in the system for reporting.                                                    |

## 6. Non-Functional Requirements
| ID      | Category          | Requirement                                                                                                |
|---------|-------------------|------------------------------------------------------------------------------------------------------------|
| **SYS-001** | **Traceability**    | All actions related to a Delivery Order (creation, status changes, cancellation) must be logged in an immutable audit trail. |
| **SYS-002** | **Interoperability**| The system shall be designed with a RESTful API to allow for future integration with other factory systems (e.g., ERP, MES). |
| **SYS-003** | **Usability**       | The user interface must be simple and responsive, designed for use on shared, potentially rugged, shopfloor terminals. |
| **SYS-004** | **Reliability**     | The system shall have an uptime of 99.5%.                                                              |

## 7. Out of Scope
- User management (creating/deleting users). Assumed to be handled by a separate IT process for v1.
- Direct integration with ERP/MES systems for inventory checks.
- Email or push notifications.
- Advanced analytics dashboard (beyond viewing the order list).

## 8. Assumptions
- User authentication is handled via a pre-existing single sign-on (SSO) system.
- All required materials have a unique SKU that is known and searchable.
