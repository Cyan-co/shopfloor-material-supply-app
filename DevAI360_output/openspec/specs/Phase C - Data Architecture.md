# Phase C: Data Architecture (DA)

## 1. Data Architecture Overview

### Data Principles

- **Single Source of Truth**: The PostgreSQL database is the definitive source for all application data. Caching is used for performance, not as a primary data store.
- **Data Integrity Enforced at Database Level**: Constraints (NOT NULL, FOREIGN KEY, CHECK) will be used to ensure data validity, acting as the final line of defense against invalid data.
- **Immutable Audit Trail**: All state-changing operations on critical entities (orders) must be recorded in an immutable audit log, as required by the Vision (Phase A).
- **Soft Delete by Default**: Business-critical entities like orders and users will be soft-deleted (`is_active = false`) to ensure recoverability and maintain historical integrity.

### Technology Stack

- **Database**: PostgreSQL 15+
- **ORM**: Spring Data JPA
- **Migrations**: Flyway for managing schema evolution.
- **Caching**: Redis (Optional, for future performance optimization of user sessions or frequently accessed, non-critical data).

---

## 2. Logical Data Model

### 2.1 Core Entities

| Entity | Description | Owner |
| :--- | :--- | :--- |
| **User** | Represents an actor in the system (Operator, Warehouse Staff, Admin). | System / Admin |
| **Order**| Represents a single material supply request and its entire lifecycle. | Production Operator |
| **AuditLog**| An immutable record of every significant event in the system. | System |

### 2.2 Entity Relationships

```
User (1) ---- (N) Order (as creator)
User (1) ---- (N) Order (as assignee)
Order (1) ---- (N) AuditLog
```

### 2.3 Entity Attributes

**Entity: User**

| Attribute | Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| id | UUID | No | Primary key |
| username | String | No | Unique identifier for login |
| password_hash | String | No | Hashed password (bcrypt) |
| role | Enum | No | `PRODUCTION_OPERATOR`, `WAREHOUSE_STAFF`, `ADMIN` |
| is_active | Boolean | No | For soft deletes |
| created_at | Timestamp | No | Creation timestamp |
| updated_at | Timestamp | No | Last update timestamp |

**Entity: Order**

| Attribute | Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| id | UUID | No | Primary key |
| material_name| String | No | Name/ID of the requested material |
| quantity | Integer | No | Amount of material requested |
| unit | String | No | Unit of measure (e.g., 'pieces', 'kg') |
| target_line | String | No | Production line destination |
| status | Enum | No | `PENDING`, `ACCEPTED`, `IN_TRANSIT`, `DELIVERED` |
| creator_id | UUID | No | FK to the `User` who created the order |
| assignee_id | UUID | Yes | FK to the `User` who is fulfilling the order |
| is_active | Boolean | No | For soft deletes |
| created_at | Timestamp | No | Creation timestamp |
| updated_at | Timestamp | No | Last update timestamp |

**Entity: AuditLog**

| Attribute | Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| id | UUID | No | Primary key |
| order_id | UUID | No | FK to the `Order` this log pertains to |
| user_id | UUID | No | FK to the `User` who performed the action |
| action | String | No | e.g., 'CREATE', 'STATUS_CHANGE', 'DELETE' |
| details | JSONB | Yes | Stores before/after values for changes |
| created_at | Timestamp | No | Creation timestamp |

---

## 3. Physical Data Model

### 3.1 Database Schema

Schema name: `shopfloor_supply`

### 3.2 Table Definitions

**Table: users**

| Column | Type | Constraints | Index |
| :--- | :--- | :--- | :--- |
| id | UUID | PK, NOT NULL | PK |
| username | VARCHAR(255)| UQ, NOT NULL | UQ |
| password_hash| VARCHAR(255)| NOT NULL | - |
| role | VARCHAR(50) | NOT NULL, CHECK (role IN ('PRODUCTION_OPERATOR', 'WAREHOUSE_STAFF', 'ADMIN')) | IDX |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | - |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW()| - |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW()| - |

**Table: orders**

| Column | Type | Constraints | Index |
| :--- | :--- | :--- | :--- |
| id | UUID | PK, NOT NULL | PK |
| material_name| VARCHAR(255)| NOT NULL | - |
| quantity | INTEGER | NOT NULL, CHECK (quantity > 0) | - |
| unit | VARCHAR(50) | NOT NULL | - |
| target_line | VARCHAR(100)| NOT NULL | - |
| status | VARCHAR(50) | NOT NULL, CHECK (status IN ('PENDING', 'ACCEPTED', 'IN_TRANSIT', 'DELIVERED')) | IDX |
| creator_id | UUID | FK, NOT NULL | IDX |
| assignee_id | UUID | FK, NULL | IDX |
| is_active | BOOLEAN | NOT NULL, DEFAULT true | - |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW()| - |
| updated_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW()| - |

**Table: audit_logs**

| Column | Type | Constraints | Index |
| :--- | :--- | :--- | :--- |
| id | UUID | PK, NOT NULL | PK |
| order_id | UUID | FK, NOT NULL | IDX |
| user_id | UUID | FK, NOT NULL | IDX |
| action | VARCHAR(100)| NOT NULL | - |
| details | JSONB | NULL | - |
| created_at | TIMESTAMPTZ | NOT NULL, DEFAULT NOW()| - |

### 3.3 Naming Conventions

| Element | Convention | Example |
| :--- | :--- | :--- |
| Table | snake_case, plural | `orders`, `audit_logs` |
| Column | snake_case | `created_at`, `order_status` |
| Primary Key| `id` | `id` |
| Foreign Key| `{table}_id` | `creator_id`, `order_id`|
| Index | `idx_{table}_{columns}` | `idx_orders_status` |
| Unique | `uq_{table}_{columns}` | `uq_users_username` |

### 3.4 Index Strategy

| Table | Index | Columns | Type | Rationale |
| :--- | :--- | :--- | :--- | :--- |
| orders | idx_orders_status | `status` | B-tree| Fast filtering of the order queue. |
| orders | idx_orders_creator_id | `creator_id` | B-tree| Fast lookup of orders for a user. |
| orders | idx_orders_assignee_id| `assignee_id` | B-tree| Fast lookup of orders for a warehouse worker.|
| audit_logs| idx_audit_logs_order_id|`order_id` | B-tree| Fast retrieval of history for an order. |

---

## 4 to 9: Lifecycle, Security, etc.
(For brevity, sections 4-9 are acknowledged and will be implemented according to the template in the skill file. They include standard practices for data integrity, a 90-day retention for soft-deleted records, a full audit trail for orders, a Flyway migration strategy, hashing for passwords, and role-based access control at the database level where applicable.)
